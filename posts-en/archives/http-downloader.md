---
id: eHK
title: Recording a Pitfall Encountered in HTTP File Download
pubDate: 2020-08-10T05:58:11.000Z
isDraft: true
tags:
  - golang
categories:
  - 笔记
---

Some time ago I ran into a requirement: we needed a downloader to synchronize data between two servers. The data is updated on a cycle of several hours, but because of deficiencies in the original system design, the file directory structures on the storage side and the application side differ, so we couldn’t use off‑the‑shelf software to handle the synchronization.

I stepped in to deal with it temporarily. Since the sync cycle and directory structure might change again later, Party A wanted this downloader to support an external configuration file so the configuration could be modified.

I immediately thought of Golang’s cobra and viper that I’d learned about before. They can quickly build a CLI application and support external config files, so I decided to implement the logic in Go.

The program logic itself is simple: the storage side publishes file services via IIS, and the application side periodically sends requests; if there is new data, it syncs.

As for how to get the file’s last updated time, my first thought was to infer it from the time information embedded in the file name. Later I was told that files with the same file name are split into forecast and actual data; previous forecast data would be overwritten by actual data, while the file name remains unchanged. So that approach had to be scrapped.

Fortunately, the default IIS directory listing page includes file modification times. So I decided to crawl the web page to collect the files whose modification times fall within the desired range.

```go
	var files map[string]string
	var f func(*html.Node)
	f = func(n *html.Node) {
		if n.Type == html.ElementNode && n.Data == "a" {

			for _, a := range n.Attr {
				if a.Key == "href" {
					if !strings.HasSuffix(a.Val, "txt") {
						break
					}
					//解析日期
					day := fmt.Sprintf("%s %s", strings.Fields(strings.TrimSpace(n.PrevSibling.Data))[0], strings.Fields(strings.TrimSpace(n.PrevSibling.Data))[1])
					//文件URL地址
					href := a.Val

					if day == "" {
						continue
					}
					if v, ok := files[day]; ok {
						files[day] = append(v, href)
					} else {
						files[day] = []string{href}
					}
					break
				}
			}
		}
		for c := n.FirstChild; c != nil; c = c.NextSibling {
			f(c)
		}
	}
	f(doc)
```

The above code crawls the IIS directory page and groups together files updated on the same date, storing them in a map. After that, I simply download the data for the desired date range to the local machine. During this process I didn’t check whether files had actually changed—any file modified within the desired date range would be re‑downloaded.

Next, I used `http.Client` to perform the requests and save the files locally.

```go
func download(url, save string) (string, error) {


	client := http.Client{}
	// Get the data
	startTime := time.Now().UnixNano()

	req, err := http.NewRequest("GET", url, nil)
	if err != nil {
		fmt.Errorf("")
	}

	resp, err := client.Do(req)
	// resp, err := Client.Get(url)
	if err != nil {
		//panic(err)
		return "", fmt.Errorf("%s", err)
	}
	defer resp.Body.Close()

	// 创建一个文件用于保存
	err = os.MkdirAll(filepath.Dir(save), os.ModePerm)
	if err != nil {

	}
	out, err := os.Create(save)
	if err != nil {
		return "", fmt.Errorf("%s", err)
	}
	defer out.Close()

	// 然后将响应流和文件流对接起来
	_, err = io.Copy(out, resp.Body)
	if err != nil {
		return "", fmt.Errorf("%s", err)
	}
	endTime := time.Now().UnixNano()
	duaring := float64(endTime - startTime)
	duaring /= 1e9

	return fmt.Sprintf("%.02fs", duaring), nil

}
```

At first I didn’t notice any problem; it was even deployed. After some time, I found that the machine where the application was deployed was frequently having network issues: unable to access the internet, or even to be accessed remotely. After all sorts of investigation, I discovered the culprit was the file downloader.

Because the update frequency was fairly high, the network connections weren’t closed immediately. As more and more ports were consumed, various network problems appeared, such as not being able to reach the external network and other applications on the server being unable to access the network.

After looking up some information, I found out the issue was that network connections were not being actively closed.

> Some servers that support HTTP/1.1 or HTTP/1.0 with the `connection: keep-alive` setting will keep a persistent connection for a period of time. But the standard library "net/http" only closes the connection by default when the server explicitly requests it, so your program may exhaust the available socket descriptors. There are two solutions; after a request finishes:
> 
> Set the request’s `Close` field to `true` directly; then the connection will be actively closed after each request.
> 
> Set the header option `Connection: close` in the request; the server’s response header will also contain this option, and at that point the HTTP standard library will proactively close the connection.[^1]

[^1]: [https://www.imooc.com/article/23959](https://www.imooc.com/article/23959)

```go
func main() {
    req, err := http.NewRequest("GET", "http://golang.org", nil)
    checkError(err)

    req.Close = true
    //req.Header.Add("Connection", "close")    // 等效的关闭方式

    resp, err := http.DefaultClient.Do(req)
    if resp != nil {
        defer resp.Body.Close()
    }
    checkError(err)

    body, err := ioutil.ReadAll(resp.Body)
    checkError(err)

    fmt.Println(string(body))
}
```

After adding this check, I confirmed that once the connection completed, the occupied ports would be automatically released.