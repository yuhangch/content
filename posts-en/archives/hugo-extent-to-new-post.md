---
id: eBI
title: Easier Article Creation for Hugo
pubDate: 2020-06-09T07:24:51.000Z
isDraft: true
tags:
  - shell
  - hugo
  - blog
categories:
  - shares
---

Idea:

- Use hugo’s `new` command with the `--editor` parameter  
- Make the new command use the `--editor` parameter by default

Take the Typora tool in a macOS environment as an example. First create `/usr/local/bin/typora` and give it execute permission:

```bash
#! /bin/bash
open -a typora $1
```

Create a new post, with a shortcut script:

```bash
#! /bin/bash
blog_path=~/Codes/blog
cd $blog_path
hugo new --editor typora posts/$1.md
```

After that, to create a new blog post, you only need to run:

```shell
blog content-title
```

In addition, for automatically adding tags and a default category when creating a new post:

Just modify `$blog_path/archetypes/default.md`.

# Deprecated

I’ve had this idea for a long time. Initially, I wanted to write a small `cli` tool in `golang` to create new posts more quickly.

After trying it today, I felt the functionality was completely unnecessary. I remembered having notes about tools like `sed`, so I decided to pick them back up and use them instead.

## Pain points

In daily use, it’s actually not that bad:

```shell
$ j blog
$ hugo new posts/new-post.md
$ typora path/to/post.md
```

Then open `Typora` and:

1. Change the auto-generated article `Title` to Chinese.  
2. Copy `Tags` and `Categories` from another article.  
3. Fill in `tags` and `categories`.

Only after completing the above steps can you start writing. The process isn’t exactly complicated, but it still feels like there’s a certain amount of repetitive work.

## Goal

My intended goal:

```shell
$ hugox new-post 新文章 随笔
```

Then `Typora` pops up and you can start writing.

## Implementation

The logic is very simple, using a script to:

1. Enter the `blog` directory.  
2. Create a new `markdown` file.  
3. Change the auto-generated English `Title` to a custom title.  
4. Add a `Tags` line.  
5. Depending on whether a category name parameter is provided, either add an empty `Categories` line or specify the category name.  
6. Open the specified file with `Typora` or `Code`.

## Portal

[Hugox](https://gist.github.com/YuhangCh/fe251512391a4f590c582ea2d566f16a)