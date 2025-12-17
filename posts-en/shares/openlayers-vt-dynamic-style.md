---
id: aFz
title: Highlighting Vector Tiles in Openlayers [Part 1]
pubDate: 2019-12-16T10:29:06.000Z
isDraft: false
tags:
  - javascript
  - openlayers
categories:
  - map
---

In a small demo I made myself, there was a requirement like this: query the database to get features with a specified attribute, then highlight them.

If you use the regular WFS loading method, there’s no problem: you can simply traverse the layer’s `feature source`. Ignoring efficiency for the moment, the logic is fine. However, there was an additional requirement: the layer has an enormous number of features (because it’s a buffer generated from global coastline data), so vector tiles were used during map loading. The vector-tile `source` type does not have a `getFeatures()` method, which makes it troublesome to traverse features to find the ones that need to be highlighted.

The static style of the layer is set using the latest `openlayers` example approach:

```javascript
//颜色表
const colorTable = {
  "No": "rgba(200, 54, 54, 0)",
  "type1": "#FF0000",
  "type2": "#E69800",
  ...
  "": "#00FFFFFF",
};
export function createRiskLevelStyle(feature) {

  const riskLevel = feature.get('props_X');
  let selected = !!selection[feature.getId()];

  return new Style({
    fill: new Fill({
      //color: colorTable[riskLevel],
      color: selected ? 'rgba(200,20,20,0.8)' : colorTable[riskLevel],
      //color:
    }),
    stroke: new Stroke({
      color: '#FFFFFF',
      width: 0.1,
    })
  })
}
```

Here `selected` is used for mouse-click highlighting. The general logic is that after a click, the feature’s `id` is stored as the key, marking that feature as selected.

Naturally, when thinking about this requirement, my first idea was to traverse the `featureCollection`, find the feature whose `Id` matches, and store it in the global `selection` variable. But because the vector-tile `source` doesn’t have a `getFeatures()` method, this idea failed. I even considered creating an additional ordinary WFS layer just to traverse the data, but the dataset is really huge—about 50+ MB per load—so that approach completely fell apart as well.

Then I thought: since the style during loading can be specified using this kind of `styleFunc`, would it work to assign a new `Func` to the layer at query time, and what about performance? So I slightly adjusted the `styleFunc` as follows:

```javascript
export function createRiskLevelStyleSearch(names) {
    return function (feature) {
        const riskLevel = feature.get('props_X')
        let properties = feature.getProperties()
        let zh = properties['label']
        let en = properties['name']
        let searched = false
        if (zh === undefined || en === undefined) {
            searched = false
        } else {
            names.forEach((v) => {
                if (en === v.key) searched = true
            })
        }
        return new Style({
            fill: new Fill({
                //color: colorTable[riskLevel],
                color: searched ? 'rgba(200,20,20,0.8)' : colorTable[riskLevel]
                //color:
            }),
            stroke: new Stroke({
                color: '#FFFFFF',
                width: 0.1
            })
        })
    }
}
```

The parameter `names` is an array of country names; each item’s `key` corresponds to the value to be queried.

With this data volume, this method works reasonably well, as shown in the figure below:

![](https://static.yuhang.ch/blog/openlayers-vt-dynamic-style_1.gif)