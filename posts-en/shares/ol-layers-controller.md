---
id: cck
title: Issues with Layer Control in Openlayers [Part 2]
pubDate: 2020-03-13T09:53:07.000Z
isDraft: false
tags:
  - javascript
  - openlayers
categories:
  - map
---

The layer control in openlayers is actually quite easy to use; you can implement it just by using the getter and setter of the layer’s `visibility`. But if you consider component binding, tree-style controls, and controlling different attribute sets within the same layer, things get more complicated.

### Binding layers and controllers

First, how to bind layers and component controllers by key–value pairs is tricky. When there aren’t many layers, you can manually add layers and bind each layer to a switch one by one. But once the layer count grows, this is no longer elegant and becomes hard to maintain.

My solution is: since layers don’t have a fixed `id`, add a `name` property when loading the layer:

```js title="ol-example.js" {2-4}
let testLayer = new VectorLayer({
    visible: true,
    name: 'test-layer',
    source: testSource
})
```

With a unique `name`, you can traverse the layers in the map, find the desired layer, and operate on it:

```js
map.getLayers().forEach((layer) => {
    let layerName = layer.values_.name;
		if (layerName !== undefined && layerName === "test-layer" {
        console.log(layer)
	}
});
```

This way, you only need to bind your custom layer name to the controller, and you can use the controller’s state to synchronize the layer’s visibility.

### Tree-style controller

A tree-style controller is different from a simple control. It has parent and child nodes: the parent node controls the visibility state of the entire layer group, while the child nodes control individual layers within that group, but are also affected by the parent node.

At first I thought about setting a flag in the `id` to distinguish child and parent nodes. Later I realized it wasn’t necessary: for several independent layers, as long as the child node states are managed correctly, the component will automatically update the parent node’s state.

For example, in the antd component I use, each `check` event produces a set of selected item IDs. This set represents the layers that should currently be visible on the map. At this point, using the previous layer traversal method, we just need to:

- Turn on any layer whose name is included in the set but is currently hidden.
- Turn off any layer whose name is not included in the set but is currently visible.

```js title="ol-example.js"
function setVisible(keyset) {
//			keyset = ['layer1','layer2',...]
        map.getLayers().forEach(function (layer) {
            let layerName = layer.values_.name;
//			Activate layers that are in the set but currently hidden
            if (layerName !== undefined && keyset.includes(layerName)) {
                if (!layer.getVisible()) {
                    layer.setVisible(true);
                }
//			Hide layers that are not in the set but currently visible
            } else if (layerName !== undefined) {
                if (layer.getVisible()) {
                    layer.setVisible(false);
                }
            }
        });
    }
```

### Controlling different attribute features within the same layer

#### For vector layers

Here I use the method of updating the `style`, similar to what was mentioned in Openlayers[1] above: assign a transparent symbol to features that are not supposed to be displayed.

Take the field `name` as an example, and let `names` be the set of attribute values for the features that should be displayed.

```js title="ol-example.js"
export function createStyleDisplay(names) {
    return function (feature) {
        const type = feature.get('name');
        const display = names.includes(type);
        if (display) {
            return new Style({
                stroke: new Stroke({
                    color: '#ffcc33',
                    width: 0
                }),
                image: new Icon({
                    //color: [113, 140, 0],
                    size: 0,
                    crossOrigin: 'anonymous',
                    //  Custom icon for the corresponding type
                    src: `/label/${type}.png`
                })
            })
        } else {
  			//... transparent style
        }
    }
}
```

#### For raster layers

I use GeoServer as the backend. For raster tile layers, you can update the request parameters.

Use `cql_filter`. GeoServer has official documentation [[link]](https://docs.geoserver.org/stable/en/user/tutorials/cql/cql_tutorial.html). For layer control, you can use the following syntax:

```text
test_field IN ('type1','type2',...,'typeN')
```

For a set of currently displayed types, here is an example of applying a change to the layer:

```js title="ol-example.js"
export function doCQL(src, field_name, keys) {
    let ql = `${field_name} IN (`
    for (let k in keys) {
        ql += `'keys[k]', `
    }
    ql = ql.substr(0, ql.length - 2)
    ql += ')'
    let pms = src.getParams()
    pms.CQL_FILTER = ql
    src.updateParams(pms)
}
```

### Postscript

Adding a `name` field in the layer properties is actually a compromise. There is no getter or setter for it; it’s just being forcefully read from the layer object. I haven’t found a method that satisfies the following: both binding the layer to a unique ID and directly obtaining the layer through that ID for operations.

Currently, to operate on layers, I need to select a certain layer from the map’s layer collection, which is not very efficient. Perhaps a hash table of `key -> layer` could be built; I may try that in the next project. If anyone has a better solution, please share it in the comments.