---
id: ass
title: Javascript Shapefile/kml/geojson Conversion
pubDate: 2021-01-16T06:23:44.000Z
isDraft: false
showTableOfContents: true
tags:
  - spatial
  - gis
  - data format
categories:
  - shares
---

## Three requirements

- geojson -> shapefile and download  
- geojson -> kml and download  
- Shapefile (zipped) -> geojson  

## geojson construction tool

Here we choose the commonly used JavaScript geometry computation library [[turfjs/turf]](https://github.com/Turfjs/turf).

Include via CDN:

```html
<script src="https://unpkg.com/@turf/turf/turf.min.js"></script>
<script>
    var bbox = turf.bbox(features)
</script>
```

Or:

```shell
npm install @turf/turf
```

```javascript
import * as turf from '@turf/turf'
```

Take a polyline as an example:

```javascript
let line_string = turf.lineString(
    [
        [-24, 63, 1],
        [-23, 60, 2],
        [-25, 65, 3],
        [-20, 69, 4]
    ],
    { name: 'line 1' }
)
let geojson_object = turf.featureCollection([line_string])
```

The printed object is as follows:

```json
{
    "type": "FeatureCollection",
    "features": [
        {
            "type": "Feature",
            "properties": {
                "name": "line 1"
            },
            "geometry": {
                "type": "LineString",
                "coordinates": [
                    [-24, 63, 1],
                    [-23, 60, 2],
                    [-25, 65, 3],
                    [-20, 69, 4]
                ]
            }
        }
    ]
}
```

## geojson to shapefile

Use the [[mapbox/shp-write]](https://github.com/mapbox/shp-write) package.

Install with npm:

```shell
npm install --save shp-write
```

Or include directly and then use the `shpwrite` variable:

```html
<script src='https://unpkg.com/shp-write@latest/shpwrite.js'>

```

The API is straightforward:

```javascript
import shpwrite from 'shp-write'

// (optional) set names for feature types and zipped folder
var options = {
    folder: 'myshapes',
    types: {
        point: 'mypoints',
        polygon: 'mypolygons',
        line: 'mylines'
    }
}
// a GeoJSON bridge for features
shpwrite.download(geojson_object, options)
```

One issue needs attention: because this package has not been maintained for a long time, using it currently will result in the following problem:

```shell
Error: This method has been removed in JSZip 3.0, please check the upgrade guide.
```

Refer to [[issue 48]](https://github.com/mapbox/shp-write/issues/48), and modify the original `shpwrite.js` file as follows:

```javascript
// ##### replace this:
var generateOptions = { compression: 'STORE' }

if (!process.browser) {
    generateOptions.type = 'nodebuffer'
}

return zip.generate(generateOptions)

// ##### with this:
var generateOptions = { compression: 'STORE', type: 'base64' }

if (!process.browser) {
    generateOptions.type = 'nodebuffer'
}

return zip.generateAsync(generateOptions)

// ##### and this:
module.exports = function (gj, options) {
    var content = zip(gj, options)
    location.href = 'data:application/zip;base64,' + content
}

// ##### with this:
module.exports = function (gj, options) {
    zip(gj, options).then(function (content) {
        location.href = 'data:application/zip;base64,' + content
    })
}
```

## geojson to kml

Use the [[mapbox/tokml]](https://github.com/mapbox/tokml) package and the [[eligray/FileSaver]](https://github.com/eligrey/FileSaver.js) file download package.

Install with npm:

```shell
npm install --save tokml file-saver
```

Include via CDN:

```html
<script src='https://unpkg.com/tokml@0.4.0/tokml.js'>
<script src='https://unpkg.com/file-saver@2.0.0-rc.2/dist/FileSaver.js'>

```

Use as follows:

```javascript
var kml_doc = tokml(geojson_object, {
    documentName: 'doc name',
    documentDescription: 'doc description'
})
var file_name = 'polyline'
var kml_file = new File([kml_doc], `${file_name}.kml`, {
    type: 'text/xml;charset=utf-8'
})
// FileSaver.saveAs()
saveAs(kml_file)
```

## Shapefile (zipped) to geojson

Use the [[calvinmetcalf/shapefile-js]](https://github.com/calvinmetcalf/shapefile-js) package; here we use a CDN as an example.

```html
<script src="https://unpkg.com/shpjs@latest/dist/shp.js">
```

```html
<!doctype html>
<html lang="en">
    <head>
        <meta charset="UTF-8" />
        <title>shapefile to geojson</title>
    </head>

    <input type="file" id="upload" />
    <script src="https://unpkg.com/shpjs@latest/dist/shp.js"></script>

    <body>
        <script>
            var Upload = document.getElementById('upload')
            Upload.onchange = function () {
                var fileList = Upload.files
                if (fileList.length < 1) {
                    return
                }
                var zip_file = fileList[0]
                zip_file.arrayBuffer().then((file) => {
                    shp(file).then((geojson) => {
                        console.log(geojson)
                    })
                })
            }
        </script>
    </body>
</html>
```