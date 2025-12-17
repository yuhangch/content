---
id: fSn
title: OL Search - A Quick Access Extension for Openlayers API
pubDate: 2022-06-16T02:00:29.000Z
isDraft: false
summary:  
tags:
  - openlayers
  - spatial
categories:
  - shares
---

## Why

The content of the openlayers API documentation is excellent, but using it is rather painful.

In general, there are two common ways to look up APIs:

- Search engine 👉 openlayers + keyword 👉 open the specific link  
- Open the API doc page 👉 search by keyword 👉 navigate to the specific result via the search results

## OL Search [^1]

OL Search is a browser extension (currently only available on Edge add-ons[^2]) that lets you quickly search the openlayers API via the browser address bar. Steps:

1. Press <kbd>control</kbd>+<kbd>L</kbd> or <kbd>cmd</kbd>+<kbd>L</kbd> to focus the address bar.
2. Type the keyword `ol`, then press <kbd>tab</kbd> or <kbd>space</kbd> to enter OL Search.
3. Enter the keyword for the target API (method, member variable, event, etc.) and select the desired link to jump directly.

## Implementation

Mainly divided into three steps:

### Parsing the API documentation

> https://openlayers.org/en/latest/apidoc/navigation.tmpl.html

The navigation part of the documentation embeds an HTML file that comes from the above address.  
There were originally two approaches.

One was to modify openlayers’ own `api build` scripts to generate a set of JSON-formatted API documentation data that matches the above HTML content. But considering two points:

- Maintenance later on: if we do this, every minor version update would require updating the extension again.
- The extension package would become larger.

The other approach is to directly parse the navigation HTML file above. This ran into a problem: in the browser extension, `backgroud.js` cannot access the `DOMParser` object. I took a detour at first: using the `popup` (the small pop-up shown when you click the extension icon) to load the data. This has a very obvious drawback: after installing the extension, users cannot use it immediately; they must click the extension icon and wait for the index file to initialize before it becomes usable. Later, I found a pure `javascript` DOM parsing library, which finally solved this problem.

### Fuzzy search

At the beginning I used a hard (exact) search. Even I was not satisfied using it myself, because occasional typos are unavoidable, so fuzzy search is essentially a must-have.

Here I referred to the approach in [mdn-search](https://github.com/hanguokai/mdn-search) and introduced `fuse.js`, plus some enhancements for multi-keyword search.

For example, when searching for the method `readFeatures`, there are methods named `readFeatures` in various formats such as `EsriJSON`, `KML`, `WKT`, etc. The default search result puts `WKT` later. If I want to find the `readFeatures` of `WKT`, that would hurt the experience.

By using `fuse.js`’s `search.$or`, I implemented composite searches with multiple keywords. Thus simply entering `readFeatures wkt` can bring the results containing `WKT` to the top of the candidates.

### Removing the default suggestion

In the callback that listens to changes in the address bar `omnibox` content, the browser will, by default, prepend a default suggestion before your custom suggestions. Its content is exactly what you typed, and the URL it points to is your extension’s address plus that content. The default behavior is effectively `File not found`.

The idea here comes from [rust-search-extension](https://github.com/huhu/rust-search-extension). First, based on the user’s input and the search results, we set the default suggestion to the original second result (i.e., the top-ranked real search result). Then, when the user presses Enter, we check whether the selected option is the default suggestion. If so, we redirect it to the URL of the actual first-ranked search result.

## Finally

I hope this tool can help all colleagues who heavily use the openlayers API docs.

[^1]: OL Search repo: https://github.com/yuhangch/ol-search  
[^2]: Edge Add-ons: https://microsoftedge.microsoft.com/addons/detail/ol-search/feooodhgjmplabaneabphdnbljlelgka