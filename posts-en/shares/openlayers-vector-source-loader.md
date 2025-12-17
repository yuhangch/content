---
id: aoV
title: Use openlayers vector source loader
pubDate: 2022-05-30T05:58:54.000Z
isDraft: false
tags:
  - gis
  - spatial
categories:
  - shares
---

The vector source's `url` option is the first choice for loading vector data, but it doesn't work when a special post-processing or loading strategy is required.

> https://openlayers.org/en/latest/apidoc/module-ol_source_Vector-VectorSource.html

## Loading strategy

First, we should learn about the loading strategy in OpenLayers. There are 3 standard loading strategies in OpenLayers:

-   `all`: load all features in a single request.
-   `bbox`: load features according to the current view's extent and resolution.
-   `tile`: load features based on a tile grid; unlike `all` or `bbox`, it takes a `TileGrid` as a parameter.

Obviously, `bbox` is the most suitable one for `loader`, because when we accept the `all` strategy, the `url` option seems fine.

## Misunderstanding

Suppose we have this requirement: our data depends on the zoom level, and when the zoom changes, we have to request data again for the current zoom level.

```javascript
...
loader:function(extent,resolution,projection){
	console.log("loading data in resolution",resolution);
	getData(resoluton).then(response=>{
		let features = source.getFormat().readFeatures(response);
		source.clear();
		source.addFeatures(features);
	})
}
...
```

In this demo code, we expect the `loader` to be triggered whenever the view's zoom changes, clear the previous features, and load new features for the new zoom. But when we scroll the mouse wheel, this is not what happens.

The log shows that `loader` is only triggered the first few times. When we keep increasing the zoom level (changing the resolution), `loader` is no longer triggered.

Why?

The extent is the main controller of `loader`. When `loader(extent...)` is called, the extent is added to the source’s loaded extent ([code](https://github.com/openlayers/openlayers/blob/main/src/ol/source/Vector.js#L1015) in Vector.js). So if the resolution changes but the new extent is still within the already loaded extent, `loader` will not be triggered.

Now it's clear: the extents from the first few calls already cover subsequent ones, so when we keep zooming in, the vector source does not invoke its loader again unless the extent exceeds the loaded extent.