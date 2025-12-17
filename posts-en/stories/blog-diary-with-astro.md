---
id: dIr
title: Blog diary with Astro
pubDate: 2024-03-27T07:57:01.262Z
tags:
  - astro
categories:
  - stories
isDraft: false
---

Checked the `git` history: the first Astro-based version of the blog was basically done around mid-July last year, and after tinkering with it through the end of 2023, it finally took shape.

From “hello world” to a few core sections, I met a lot of new friends along the way. Through those conversations I gained many new ideas about blogs and about building one.

## Why I chose Astro

After saying goodbye to [Hugo](https://gohugo.io/), my first choice was actually [Nuxt](https://nuxt.com/). I even wrote a post about it: [Some thoughts on writing a blog with Nuxt](https://www.v2ex.com/t/952490).  

The reason I left [Hugo](https://gohugo.io/) was that while the templating language keeps builds efficient, it’s really painful to heavily customize, and at that time I couldn’t find any good IDE plugins. I had the idea of writing a [Hugo](https://gohugo.io/) theme from scratch more than once, but each attempt ended in failure.

In that post I wrote that the positive feedback loop from [Nuxt](https://nuxt.com/) during development was almost addictive, but there were a few issues:

1. A clean, beautiful HTML page becomes very cluttered in the page source once it’s wrapped in all kinds of Vue components.
2. It’s an SPA (nothing wrong with that, I just don’t like it much).

In the end I was “intimidated” into trying [Astro](https://astro.build/) by their [2023 Web Framework Performance Report](https://astro.build/blog/2023-web-framework-performance-report/).

After trying it, both the docs and the IDE development experience felt fine, so I officially fell into the pit. Back then [Astro](https://astro.build/) was still iterating rapidly, which also catered to my urge to tinker.

## Why I like bearblog

A long time ago, while looking something up, I found this post: [Mike - Line Simplification](https://bost.ocks.org/mike/simplify/). I “stole” the minimalist and elegant page design from there, and also learned D3.js from it. It was a big win.

Later, I saw bearblog on [Hacker News - Bear Blog – A privacy-first, fast blogging platform](https://news.ycombinator.com/item?id=32127363) and fell in love at first sight. My previous Hugo blog was also based on a bearblog theme.

The minimalist page design and native web buttons really hit home for me. I try to keep only content on the page, even hiding the navigation bar when possible—all inspired by bearblog.

## What I did for simplicity

I also wrote a post about this: [Suddenly tailwindcss doesn’t feel that great anymore](https://www.v2ex.com/t/994735#reply59).

> I used astro to build a static site, mainly text-based.  
> At the time I used tw mainly for productivity: light/dark mode toggling, prose typography, etc.  
> Now that most features are done and I want to optimize, I checked one article’s index.html:  
> total size ~78kb, but after removing all tw variable declarations and class definitions it’s only ~24kb…  
> I tried purgecss, but it didn’t help much (maybe I used it wrong)?

I wanted the convenience of tailwindcss but didn’t want to pay such a huge price (almost triple the size). A V2EX friend suggested [Unocss](https://unocss.dev/).

Migration wasn’t hard, and its usage is basically compatible with tailwindcss. Looking back, my homepage single page is only `11kb` now, so I still feel that this round of tinkering was worth it.

## My preferred way of organizing i18n

When I was working on i18n, I kept wavering. On one hand, it’s quite a bit of effort. On the other, I kept asking myself: “Are there really any native English speakers reading my blog?”

In the end, my love of tinkering beat my “lack of confidence,” and I implemented it anyway.

Although `Astro` now has built-in i18n support, I’m still using the third-party plugin [astro-i18n-aut](https://github.com/jlarmstrongiv/astro-i18n-aut).

The reason is that the official i18n requires organizing both components and content into different directories per language, and I don’t like that design.

My i18n implementation roughly has two parts. First, localized text is stored in `YAML`. This does mean adding one more package, `@rollup/plugin-yaml`, but I really dislike `JSON`.

```yaml title="zh.yml"
layout:
    title: 陈昱行博客
    description: 陈昱行的个人博客，在没人看到的地方写写画画。少些技术，多些生活。这里自己的学习生活，偶尔分享一下自己对这个世界的看法。
nav:
    posts: 随笔
footer:
    home: 首页
    rss: 订阅
    timeline: 时间线
```

The second part is the content itself. Knowing I’d be lazy sometimes, I can’t guarantee that all content will exist in two languages, but I still want all content to be visible on both the Chinese and English interfaces.

So I came up with this design: use extended file extensions to distinguish Chinese and English. English content ends with `.en.md` or `.en.mdx`. If a corresponding file exists, show it; otherwise fall back to the Chinese version.

I also enforce an English title in the frontmatter, so at least the homepage can be fully in English, which looks nicer.

## From pseudo-dynamic to real dynamic: “Moments”

The “Moments” section originated from [叽喳](https://www.giki.app/zh-CN). Later, because it was a bit unstable and I wanted control over the data, I built something similar myself: [live](https://github.com/yuhangch/live).

This time, taking the opportunity of refactoring the blog, I integrated that part as well.

At the beginning, I considered two options:

1. Use leancloud to store data, as before.
2. Make this part static too, storing it in a text file.

After some research, I found `cloudflare D1`. The good news is that it’s available on the `Free Plan`, so I stopped agonizing over the choice.

```sql title="create-moment.sql"
DROP TABLE IF EXISTS moments;
CREATE TABLE IF NOT EXISTS moments (
  id integer PRIMARY KEY AUTOINCREMENT,
  body text NOT NULL,
  tags text WITH NULL,
  star integer NOT NULL default 0,
  created_at text NOT NULL
  deleted_at text WITH NULL
);
CREATE INDEX idx_moments_created_at ON moments (created_at);
```

I used a `cloudflare worker` to create a simple API to fetch data.

On the client side, I fetch data via SSR and paginate the rendering. The upside is that I don’t expose the API publicly. The downside is that if someone wants to “hammer” an endpoint, they can just keep requesting the page and there’s not much I can do.

## TMDB-based movie reviews

My original intention was to use imdb or Douban: on one hand I could reuse their clients, on the other they’re very comprehensive.

But their anti-scraping measures are so strict that there was simply no way in. I even tried using a headless browser approach to access IMDB’s `rating` CSV export endpoint; that failed too. So I decided to go with TMDB.

Maintaining my own data also has the advantage that I can integrate it with my “Moments”, linking each review to a specific “Moment” entry.

```sql title="create-review.sql"
DROP TABLE IF EXISTS reviews;
CREATE TABLE IF NOT EXISTS reviews (
	id integer PRIMARY KEY AUTOINCREMENT,
	imdb_id text NOT NULL,
	title text NOT NULL,
	title_en text NOT NULL,
	media_type text NOT NULL,
	imdb_rating real NOT NULL,
	rating real NOT NULL,
	release_date text NOT NULL,
	rated_date text NOT NULL,
	moments_id integer NOT NULL,
	created_at text NOT NULL,
	deleted_at text WITH NULL
);
```

## Short links

This feature is more form than substance, but better than nothing. The rough idea:

1. Add a hook at build-completion time to create new short links for URLs not yet in the database.
2. Add an [id] route to handle short links.

### uuid

Each short link is composed of a fixed prefix and three random characters, which is more than enough for my needs.

- Prefix [b] means a blog link.
- Prefix [o] means a notebook/draft link.

```typescript title="uuid.ts"
import shortUUID from "short-uuid";

const characters = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ";
const short3 = shortUUID(characters);

export function getOpenUUID(exist: string[]) {
    // generate a v4 uuid, subtract the first 3 characters, and then convert to base62
    let uuid = 'o' + short3.generate().slice(0, 3);
    while (exist.includes(uuid)) {
        uuid = 'o' + short3.generate().slice(0, 3);
    }
    return uuid
}
```

### Persisting short links

I hesitated for a long time here. Using `vercel kv` directly has limited free monthly quota, so I ended up combining it with `cloudflare D1`:

```sql title="create-short.sql"
DROP TABLE IF EXISTS shorts;
CREATE TABLE IF NOT EXISTS shorts (
  id text PRIMARY KEY NOT NULL,
  url text NOT NULL,
  created_at text NOT NULL,
  deleted_at text WITH NULL
);
CREATE INDEX idx_shorts_id ON shorts (id);
```

### Building the short-link route

With everything in place, all that’s left is a route to parse the short link.

```javascript title="[id].js"
import { createClient, kv as prodKV } from '@vercel/kv'

const notFound = new Response(null, {
    status: 404,
    statusText: 'Not found'
})
const getLink = async (id) => {
    const kv = DEV
        ? createClient({
            url: KV_REST_API_URL,
            token: KV_REST_API_TOKEN
        })
        : prodKV
    const cached = await kv.get(id)
    if (cached) {
        return cached
    }
    const apiURL = `api.blog.com`
    const headers = {
        Authorization: `Bearer ${BLOG_API_SECRET}`
    }
    const res = await fetch(apiURL, { headers })
    if (res.status === 404) {
        return null
    }
    const json = await res.json()
    const url = json.url
    await kv.set(id, url)
    return url
}
export const GET = async ({ params, redirect }) => {
    const { id } = params
    const type = id?.slice(0, 1)
    if (!type || !['o', 'b'].includes(type)) {
        return notFound
    }
    const link = await getLink(id)
    if (!link) {
        return notFound
    }
    return redirect(link, 308)
}
```

## Code blocks

This also went through a few iterations. At first I liked [code-hike](https://codehike.org/docs/introduction), but because of [#255](https://github.com/code-hike/codehike/issues/255), I never actually used it.

Later I followed [Highlight a line on code block with Astro](https://sat0shi.dev/posts/highlight-line-on-codeblock-with-astro/) and migrated some styles. It looked pretty good, but my own styling got a bit complicated and I felt it was polluting the CSS.

Finally I came across [starlight - expressive code](https://starlight.astro.build/reference/configuration/#expressivecode), which solved almost all my pain points, and copying over the styles worked just fine.

```typescript title="astro.config.mts"
import expressiveCode from 'astro-expressive-code'
import {ExpressiveCodeTheme} from '@expressive-code/core'
import {readFileSync} from 'fs'
import {parse} from 'jsonc-parser'

const nightOwlDark = new ExpressiveCodeTheme(
    parse(readFileSync('./src/styles/expressive-code/night-owl-dark.jsonc', 'utf-8'))
)
const nightOwlLight = new ExpressiveCodeTheme(
    parse(readFileSync('./src/styles/expressive-code/night-owl-light.jsonc', 'utf-8'))
)

// 插件配置
···
expressiveCode({
    themes: [nightOwlDark, nightOwlLight],
    themeCssSelector: (theme) => {
        return '.' + theme.type
    }
})
···
```

## component or directive?

At first I added hints using a component, but that meant I had to convert `md` to `mdx` just for this, which felt too costly. Later I switched to using [remark-directive](https://github.com/remarkjs/remark-directive).

:::note{.info}
Tip: this is a tip
:::

The usage is as follows; you can just refer to the examples in `remark-directive`.

```markdown title="notification.md"
:::note{.info}
提示：这是个提示
:::
```

In addition, I had a post that needed to embed a Bilibili video, so I implemented that via `remark-directive` as well.

```javascript title="bilibili.js"
export function RDBilibiliPlugin() {
    return (tree, file) => {
        visit(tree, function (node) {
            if (
                node.type === 'containerDirective' ||
                node.type === 'leafDirective'
            ) {
                if (node.name !== 'bilibili') return
                const data = node.data || (node.data = {})
                const attributes = node.attributes || {}
                const bvid = attributes.id
                if (!bvid) {
                    file.fail('Unexpected missing `id` on `youtube` directive', node)
                }
                data.hName = 'iframe'
                //<iframe src="//player.bilibili.com/player.html?bvid=BV1Zh411M7P7&autoplay=0" width="100%" allowfullscreen> </iframe>
                data.hProperties = {
                    src: `//player.bilibili.com/player.html?bvid=${bvid}&autoplay=0`,
                    width: '100%',
                    height: 400,
                    aspectRatio: '16 / 9',
                    // fit height
                    class: 'm-auto',
                    // height: 400,
                    frameBorder: 0,
                    allow: 'picture-in-picture',
                    allowFullScreen: true
                }
            }
        })
    }
}
```

## Why there are no comments

I once saw a blogger whose comment section was just a big “Please contact me by email”, which I thought was very cool.

On the other hand, there’s a question I haven’t figured out: I don’t like anonymous comments, but if I require authentication, whether via GitHub or some other third-party login, it still won’t cover all readers.

So if you have something to say, just send me an email instead ღ( ´･ᴗ･` )比心

## From blog framework to personal skill

Since I first tried Astro in order to build a blog, my feelings have gone from “I kind of like it” to choosing it as my preferred static site generator. It all feels like a natural progression.

- Personal page: [chenyuhang.com](https://chenyuhang.com)
- Writings:  [chenyuhang.cn](https://chenyuhang.cn)
- Notebook:  [open.yuhang.ch](https://open.yuhang.ch)