---
id: auP
title: Those little tricks you wish you’d known sooner
pubDate: 2020-04-11T05:07:37.000Z
isDraft: true
tags:
  - OS X
  - Shell
  - Softwares
categories:
  - 技巧
---

# Mac-related

## Switch between windows of the same app

- ⌘ + _`_

## Spotlight Search

- ⌘ + ␣

This is the default key binding. I didn’t use it often before because I was used to doing shortcuts with my left hand only.

On a whim I tried left-hand `cmd` plus right-hand `space`, and it felt much more natural. Maybe this will inspire anyone who has also only been using their left hand for shortcuts.

## Pinning tabs in Safari

Drag a tab to the left to pin it. Only the icon will be shown, which is suitable for frequently used tabs.

# Git-related

## stash/commit/push in one step

```bash
git config --global alias.cmp '!f() { git add -A && git commit -m "$@" && git push; }; f'
```