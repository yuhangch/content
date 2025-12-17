---
id: esG
title: helix editor
pubDate: 2022-04-25T03:13:00.000Z
tags:
  - editor
  - vim
categories:
  - shares
---

> helix editor : https://helix-editor.com/

A command-line vim-like editor written in Rust (there’s a simple demo video above 👆 so I won’t screenshot it). I came across it on YouTube a few days ago, tried it for a bit, and it has both sweet spots and pain points.

It calls itself “post-modern,” which feels more like a jab at those editors that call themselves “modern.”

As for “vim-like”, the keybindings are inherited from Vim and Kakoune. If you know Vim you can jump right in (most familiar commands also work, like `:vs`), but the operational logic is different, which is both satisfying and painful:

For example, if you want to `dd`, `V`, you’ll feel uncomfortable: in helix `x` selects the line, and `d` can replace the function of `x`. In helix, `w`, `b`, etc. will select text by default, so `dw` becomes `wd`.

As for Multiple selections, I haven’t really used them in other editors before, so I can’t comment much on that experience. (Is it similar to holding option and dragging in IDEA? If so, then it’s indeed quite handy.)

Now the sweet spots:

Compared to vscode, you can start it directly from the command line, without a `code .` type window popping up.

Compared to vim/nvim, you don’t have to think about XXX-complete, XXX-line, fzf or leaderf — helix provides a full package of support.

The built-in file-picker and buffer-picker design really fits my taste: not flashy, just simple and sufficient.

LSP and tree-sitter support are solid; configuring LSP for json and toml, which I edit frequently, is simple. I tried writing Rust with rust-analyzer and it was actually quite nice. (I still choose an IDE though 🙃️)

The basic features are restrained, sufficient, and easy to use, but on the other hand there’s almost no extensibility. I didn’t see anything like extension/plugin mentioned in the docs.

For me, I used to mainly use vscode to edit simple text; going forward helix will probably be my first choice for editing simple text. But the pain points are real too: some differences in keybindings from Vim can be mentally jarring:

```txt
dd uu xd
```

So I went back to nvim and tried to configure helix-style file-picker and buffer-picker (then gave up — it’s not like I can’t edit text without them).

By the way, does anyone know of a similar editor?

-   Vim-support, not LIKE  
-   Built-in language server support.  
-   Syntax highlighting and code editing using Tree-sitter.  
-   Built with XXX. No Electron. No VimScript. No JavaScript.  
-   Runs in a terminal.