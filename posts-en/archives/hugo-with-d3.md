---
id: avb
title: hugo integrate d3.js/Observable Notebook
pubDate: 2020-08-11T13:49:02.000Z
isDraft: true
tags:
  - hugo
  - blog
categories:
  - archive
---

> The following image is from https://observablehq.com/@d3/zoom-to-bounding-box

I first came across a blog post, [Line Simplify](https://bost.ocks.org/mike/simplify/), and originally thought it was just a plain loaded image.

After clicking around on the screen I realized the map wasn’t a rendered image at all, but directly rendered SVG. The effect is great, and the clarity is excellent, so I started wondering if I could port it into my own blog posts.

After an Inspect, I found the tool being used was d3.js, and by following the trail I discovered the powerful [observable notebook](https://observablehq.com/).

I googled a bit to see whether integration was possible, and a post by Jeremy suggested using a shortcode for integration.[^1]

According to the official documentation,[^2] you can easily obtain the `Embed Code`, as in the following example:

```html
<div id="observablehq-9415c34d"></div>
<script type="module">
    import { Runtime, Inspector } from 'https://cdn.jsdelivr.net/npm/@observablehq/runtime@4/dist/runtime.js'
    import define from 'https://api.observablehq.com/@gengtianuiowa/homework-2.js?v=3'
    const inspect = Inspector.into('#observablehq-9415c34d')
    new Runtime().module(define, (name) => (name === 'choropleth' ? inspect() : undefined))
</script>
```

In theory, integrating the above code into the page is all that’s needed to display it.

## Create a new shortcode template

Create a new template file: in `layout/shortcodes/observablenotebook.html` or `themes/yourtheme/layout/shortcodes/observablenotebook.html`

Analyzing the `Embed Code`, the only variables we need to set are the `id` of the `<div>` and the code block to be rendered.

```html
<div id="observablehq-{{.Get 0}}"></div>
<script type="module">
    import { Runtime, Inspector } from 'https://cdn.jsdelivr.net/npm/@observablehq/runtime@4/dist/runtime.js'
    import define from '{{.Get 1}}'
    const inspect = Inspector.into('#observablehq-{{.Get 0}}')
    new Runtime().module(define, (name) => (name === 'chart' ? inspect() : undefined))
</script>
```

## Import the notebook.js file

```markdown
---
title: 'Integrating d3.js/Observable Notebook in hugo'
pubDate: 2020-08-11T21:49:02+08:00
isDraft: false
tags: ['hugo']
categories: ['Notes']
---

> The following image is from https://observablehq.com/@gengtianuiowa/homework-2
```

[^1]: [Embed an Observable Notebook in a Hugo site](https://kinson.io/post/embed-observable-notebook/)
[^2]: [Downloading and Embedding Notebooks](https://observablehq.com/@observablehq/downloading-and-embedding-notebooks)