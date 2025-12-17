---
id: aTR
title: An horizontal-pan-only map in openlayers
pubDate: 2022-05-12T03:42:02.000Z
tags:
  - map
categories:
  - shares
---

We usually use 2D maps. In fact, there are many available web map libraries such as [openlayers](https://openlayers.org), [leaflet](https://leafletjs.com), or [mapbox-gl-js](https://docs.mapbox.com/mapbox-gl-js/). I’ll introduce a way to make a horizontally constrained map in openlayers.

To make the map move only horizontally, we have to hook into these interactions: `pan` and `wheel scroll zoom`.

The default interactions that openlayers uses for these can be found at the following links:

- dragPan: [DragPan.js](https://github.com/openlayers/openlayers/blob/main/src/ol/interaction/DragPan.js)  
- mouseWheelZoom: [MouseWheelZoom.js](https://github.com/openlayers/openlayers/blob/main/src/ol/interaction/MouseWheelZoom.js)

## Disable default interaction

First, we disable the map’s default interactions.

```javascript
const map = new Map({
  ...
  interactions: defaultInteractions({
    dragPan: false,
    mouseWheelZoom: false,
    doubleClickZoom: false
  })
  ...
}
```

After applying this option, the map can no longer be controlled, which is what we expect at this step.

## Hook interaction

### Drag pan

First we create a custom pan interaction extending from `DragPan`.  
The default interaction implements three methods to handle the `Drag Event`, `Pointer Up`, and `Pointer Down` events. The `Drag Event` handler contains the [coordinate computation](https://github.com/openlayers/openlayers/blob/main/src/ol/interaction/DragPan.js#L102-L105). In other words, we need to override `handleDragEvent`.

```javascript
class Drag extends DragPan {
  constructor() {
    super();
    this.handleDragEvent = function (mapBrowserEvent) {
      ...
          const delta = [
            this.lastCentroid[0] - centroid[0],
            // centroid[1] - this.lastCentroid[1],
            0
          ];
     ...
    }
```

The second element of `centroid` stores the y coordinate, so we comment out the line that calculates the y delta and set it to zero.

```javascript
const map = new Map({
...
interactions: defaultInteractions({
  dragPan: false,
  mouseWheelZoom: false,
  doubleClickZoom: false
}).extend([new Drag()]),
...
})
```

Add the custom drag interaction after the `defaultInteractions` function, and now our map can be panned with mouse drag, but only horizontally.

### Mouse wheel zoom

Following the drag pan section, we can easily find the coordinate computation in `MouseWheelZoom`.  
It appears at [L187-L189](https://github.com/openlayers/openlayers/blob/main/src/ol/interaction/MouseWheelZoom.js#L187-L189). Do a small tweak in the `handleEvent` method:

```javascript
const coordinate = mapBrowserEvent.coordinate
const horizontalCoordinate = [coordinate[0], 0]
this.lastAnchor_ = horizontalCoordinate
```

Just like with `dragPan`, we add a custom `MouseWheelZoom` interaction `Zoom` after the default interactions.

```javascript
const map = new Map({
...
interactions: defaultInteractions({
  dragPan: false,
  mouseWheelZoom: false,
  doubleClickZoom: false
}).extend([new Drag(),new Zoom]),
...
})
```

Now our map can zoom using the mouse wheel, and it only works in the horizontal direction.