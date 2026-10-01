---
title: "我好像陷入了no webview的原教旨主义"
pubDate: 2026-10-01T14:19:54+08:00
categories: ["thought"]
isDraft: false
---

最近做了俩软件，[pandanote](https://github.com/panda-note/panda-note)和[pandareader](https://github.com/yuhangch/panda-reader)，note是自己看到[evernote](https://evernote.com/)做的，reader是看到[papr](https://github.com/l0ng-ai/papr)做的，原因是他俩是webview，我想用，而我“不太喜欢”webview。

写这篇文章是在做reader的时候内心一直处在一种拧巴的状态，虽然自己做也挺方便的，但看着界面和ai聊天也是种消耗。想通过写篇东西顺道梳理一下自己。我在构思pandareader的app story的时候，就明确了这个软件就是我自己用的：

> Choosing a reader started taking as much time as making one. So I made Panda Reader. I can change it whenever I like. That’s the whole advantage.
>
> It won’t suit everyone. Use it, change it, fork it. That’s why it’s open source.

1. 现在挑选一个产品和自己手写一个的时间成本差不多
2. 这个产品没有更多的title，唯一的优点是我想怎么改就怎么改
3. 开源出来没想过有人直接会用，把它作为一个起始点开发自己的产品他就是有意义的

自己写着写着意识到，并非只是我这个产品，而是这类终端应用某种意义上都要死了。

## 是不是webview真的重要吗

事实上，我每天使用时间最长的软件都离不开webview：[codex](https://openai.com/codex/)、[cursor](https://cursor.com/)，是我的生产力工具也让我备受折磨，两个软件跑在我工作站上时不时卡一下是常态。我喜欢[zed](https://zed.dev/)但讽刺的是我工作时间并不经常使用，因为zed的agent不支持workspace的多目录，而我工作往往是跨项目的（这个问题已经有几个月了）。

前两年大家都在从[electron](https://www.electronjs.org/)往[tauri](https://tauri.app/)上转，
tauri已经解决了软件体积的问题，所以大多数人对一个应用是不是用的web技术没那么关心了。剖析下自己，我可能是更喜欢某种设计风格，和原生ui应该有更高的性能所带来的情绪价值，也可能是对web应用里各种AI做的大大小小的圆角样式有些厌倦。

whatever，感谢[zed](https://zed.dev/)开源了[gpui](https://github.com/zed-industries/zed/tree/main/crates/gpui)，感谢长桥开源了[gpui-kit](https://github.com/longbridge/gpui-kit)，让我的偏好有的放矢。

## 终端应用客观上就满足不了所有人

想到一个比喻，传统的软件像固定尺码的西装，用于解决某类问题，由业务引出，再由公司投入一定的人力物力将其做成标品，用户再去适应这个和自己身形差不多，但总没那么合身的衣服，人之所以能忍是找裁缝做时间和金钱成本太高，而生成式ai是给每个人没那么贵的赛博裁缝角色。

这个比喻做了一种简化，软件的细节比服装之前的差异大的多，从技术栈、用户界面、数据是封闭的还是开放的，当前vibe coding发展下，是有越来越的选择，但找的一个自己各方面都契合的并不容易。

我特别喜欢[obsidian](https://obsidian.md/)，笔记软件社区插件生态断档领先，很好用的vim模式，但其移动端不好用，第三方同步体验不佳；此外，每次看到有朋友截图自己的笔记，用数字对文件夹进行排序的时候就很想笑:)，为了适应file管理模式，这代价是不是太大了。

因此所有的终端软件客观上都不是适合所有人的，过去大家能接受实际上也是在妥协。

## 有价值的开源是各类基础设施

开箱即用是美好的设想，但在大家都有了自己的赛博裁缝之后，这些产品即使是开源的产品的价值都已大打折扣。这种开箱即用的服务于作者个人的产品，之后最大的价值也只是作为一个起始点供大家继续。而作为作者也没必要设想这个产品可以服务大多数人。

与此相反的是，基础设施、框架的价值正在不断放大。一个比较直观的现象是，大家目前还是在用[Astro](https://astro.build/)或者[hugo](https://gohugo.io/)这样的框架，但相较于去主题商店找主题，更常见的是根据自己的想法在做服务于自己的主题。

再例如[gpui](https://github.com/zed-industries/zed/tree/main/crates/gpui)，现在发展势头迅猛，很多的套壳的应用正在向此转向，大家苦webview久矣，但之前都拿他没有办法，不过我好像已经看到开发者们在举着大旗向其冲锋了。

还想了另一个标题：独立开发者好像要死了，但我这不是公众号，好像没必要有这么“爆”的标题。
