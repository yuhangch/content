---
id: fRW
title: Blog Creation Tool
pubDate: 2023-07-26T08:16:21.386Z
tags:
  - javascript
categories:
  - shares
isDraft: false
---

## In short

Compared with blog platforms like WordPress, using a Git-based / Markdown workflow to manage your own content is undoubtedly more controllable and more flexible for cross‑platform migration, but it also increases the mental overhead of creating a post. Even though there are CLI tools like `hugo new`, categories and tags still have to be manually edited in the frontmatters.

I previously tried something similar and wrote a simple tool using `fish shell`.

```fish showLineNumbers {1-3} title="functions/blog.fish"
function blog
    cd /dest/to/blog
    set -Ux BLOG_SLUG $argv[2]

    set c $argv[1]
    argparse t/tag -- $argv
    or return

    set md (string split '"' (hugo new posts/$c/$argv[2].md) -f2)
    sed -i '' 5s/\"\"/\"$c\"/ $md
    if set -ql _flag_tag
        sed -i '' 6s/\"\"/\"$argv[3]\"/ $md
    end
    echo $md
    open $md

end
```

The idea is still to create an article via a slug, list out existing categories beforehand, and then complete them with Tab. This is a bit more convenient than editing in frontmatters directly.

```shell title="completes/blog.fish"
complete -c blog  -xa "comments  drafts  shares  stories  thoughs  thoughts  translations " -n '__fish_is_first_arg'
```

Recently I rebuilt the blog using astro, added `i18n` support and short links, and other features. This requires more frontmatter fields such as subtitles and IDs, so the script needed to be modified. However, maintaining a `fish` script is a bit troublesome, so I decided to rewrite it as a `node` script.

## New feature requirements

-   An interactive CLI tool
-   Translate titles via online translation or by calling an LLM
-   Auto‑generate short links, and generate slugs from English titles
-   Derive categories and tags from existing posts
-   Image management

## Implementation path

### Implementing an interactive CLI

Tech stack choices:

-   [Inquirer.js](https://www.npmjs.com/package/inquirer)
-   [commander](https://www.npmjs.com/package/commander)

`inquirer` is used for interactive prompts, and `commander` for parsing CLI arguments. Initially I planned for two commands.

1. create – create a post  
2. upload – upload an image, with one option `--file -f` to determine whether to upload a local file or an image from the clipboard  
3. activate – switch the currently edited article (similar to conda environment management)  
4. translate – translate a specified article `//TODO`

### Implementing automatic title translation

After some experiments, direct machine translation performed poorly, so I ended up using openai-3.5-turbo (azure). Based on the input title, I first determine whether it’s Chinese or English, then give the opposite‑language prompt and let the model translate the title.

```json
{
    "messages": [
        {
            "role": "system",
            "content": "你是一个中英翻译助手，将下面这个博客文章的标题翻译成${dest}文。"
        },
        { "role": "user", "content": "${title}" }
    ]
}
```

This way, regardless of whether the original title is Chinese or English, you only need to input it once and let the AI handle the other language.

### Implementing automatic short‑link generation

Since the number of my posts won’t be very large, I use three case‑sensitive letters to generate short links. This is enough to ensure uniqueness while keeping the links short. I use the [short-uuid](https://www.npmjs.com/package/short-uuid) package to generate UUIDs, but only take the first three characters, then check if that ID already exists; if it does, generate another.

```javascript
const chars = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ'
const translator = uuid(chars)
let id = translator.new().slice(0, 3)
while (keyExits(id)) {
    id = translator.new().slice(0, 3)
}
```

Short links are ultimately implemented using Vercel’s [redirects](https://vercel.com/docs/concepts/projects/project-configuration#redirect-object-definition). This way, you only need to update `vercel.json` in the project root and redeploy.

```json
{
    "redirects": [{ "source": "/${id}", "destination": "/posts/${slug}" }]
}
```

### Image management

Why not use existing image‑hosting tools?

1. They can’t fully fit into my current workflow  
2. Their file‑naming conventions don’t match my preferences  

So I added one more step here: when generating a post, I store a global environment variable so that when managing images, I can operate in the folder generated from that post’s slug.

-   Existing image files are saved in a specified directory, then uploaded to object storage via a command.
-   Images from the clipboard are first saved locally via [pngpaste](https://formulae.brew.sh/formula/pngpaste), then uploaded to object storage.

Finally, I use `pbcopy` to push the image URL directly into the clipboard, so it can be pasted into the post, like this:

```shell
blog upload test-jpeg
URL: https://static.yuhang.ch/blog/blog-creation-tool/test-jpeg.jpeg
```

For a screenshot in the clipboard, in this workflow you only need to think of a slug for it; the tool uploads it to the designated directory in object storage and puts the link in your clipboard.

## Ending

This article was created with this tool.