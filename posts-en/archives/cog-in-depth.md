---
id: bev
title: Cloud Optimized GeoTIFF (COG) In Depth
pubDate: 2020-02-24T03:36:49.000Z
isDraft: false
tags:
  - cog
  - data format
categories:
  - translation
---

## Overview

Cloud Optimized GeoTIFF (COG) relies on two auxiliary technologies:

- The first is GeoTIFF’s storage capabilities: storing pixels in a special way, rather than just storing raw pixels directly.

- The second is HTTP GET range requests, which allow a client to request only the parts of a file it needs.

The first, GeoTIFF’s storage layout, makes it convenient for the second to retrieve only the parts of the file that need to be processed.

## GeoTIFF Organization

The two main data organization techniques used by COG are tiling and overviews; compression further improves the efficiency of data transfer over the network.

**Tiling** creates internal tiles in the imagery instead of simply using data strips. With strip-based storage, retrieving specific data requires reading the entire strip. Once tiles can be quickly accessed over specified areas, the same request can be fulfilled by accessing only a certain portion of the data.

**Overviews** create multiple downsampled versions of the same image. Downsampling means that when you “zoom out” from the original image, many details disappear (a single current pixel may correspond to 100 or even 1000 pixels in the original), and the data volume becomes smaller. Typically, a GeoTIFF will have several overviews to match different zoom levels. This speeds up server responses, because rendering only requires returning those specific pixel values instead of figuring out which pixel should represent those 1000 pixels. However, this also makes the overall file size larger.

Compression allows software to quickly retrieve the imagery and usually gives a better user experience, but making HTTP GET range requests efficient remains very important.

## HTTP GET Range Requests

HTTP/1.1 introduced a very powerful feature: range requests, used in GET requests when a client requests data from a server. If the server’s response headers contain `Accept-Ranges: bytes`, it indicates that the bytes of the data can be requested by the client in arbitrary chunks. This is often called “Byte Serving”; Wikipedia has an [article](https://en.wikipedia.org/wiki/Byte_serving) explaining how it works in detail. Clients can request only the bytes they need from the server. In the web world, this is widely used, for example in video services, so that clients can operate on the file without downloading it in full.

Range requests are an *optional* feature, so servers are not required to implement them. However, most cloud providers’ (Amazon, Google, Microsoft, OpenStack, etc.) object storage tools provide this option. As a result, most data stored in the cloud already supports range requests.

## Putting It Together

With these two technologies introduced, it becomes clear how they work together. Tiling and overviews in a GeoTIFF are stored in a well-defined structure within the file in cloud storage, so range requests can target the relevant parts of the file.

Overviews are used when a client wants to render a quick view of an entire image without downloading every pixel. The request is transformed into a request for the smaller, pre-generated overview. The specific structure of the GeoTIFF file, combined with a server that supports HTTP range requests, lets clients easily obtain just the portions of the file they need.

Tiles are useful when only parts of an image need to be processed or visualized. This could be a part of an overview or part of the full-resolution data. Note that tiling organizes all data for a given area together at the same location in the file, so range requests can fetch it on-demand.

If a GeoTIFF has not been “cloud optimized” with overviews and tiles, remote operations are still possible, but they require downloading the entire dataset or downloading far more data than is actually needed.

## Advantages

More and more geospatial data is moving to the cloud ☁️, and most of it is stored in cloud-based object storage such as [S3](https://aws.amazon.com/s3/) or [Google Cloud Storage](https://cloud.google.com/storage/). Traditional GIS file formats can be easily stored in the cloud, but when it comes to providing web map tile services or performing fast data processing, these formats lose efficiency. Typically, the data must be fully downloaded elsewhere, then converted into a more optimized format or read into memory.

Cloud Optimized GeoTIFF uses some [small tricks](https://www.cogeo.org/in-depth.html) to make data streaming more efficient and to enable cloud-based geospatial data workflows. Online imagery platforms such as the [Planet Platform](https://www.planet.com/explorer/) and [GBDX](https://platform.digitalglobe.com/gbdx/) use this approach to provide imagery services, enabling very fast imagery processing. Software that supports COG can optimize execution time by fetching only the portions of data it needs.

Many newer geospatial software projects such as [GeoTrellis](http://geotrellis.io/), [Google Earth Engine](http://earthengine.google.com/), and [IDAHO](https://gbdxdocs.digitalglobe.com/docs/idaho-course) also incorporate COG concepts in their architectures. Each processing node performs high-speed image processing by fetching partial file streams from COGs.

As for the impact on the existing GeoTIFF standard, this is not like introducing a brand new file format. Existing software can read COGs without any changes. They do not need to support streaming; they can simply download the entire file and read it as usual.

Providing files in Cloud Optimized GeoTIFF format in the cloud helps reduce a large number of file copies. Because online software can use streamed access instead of keeping its own copy, this becomes more efficient and is also a common pattern today. In addition, data providers do not need to supply multiple formats, because both older and newer software can read the same data. Data providers only need to update a single version of the data, and at the same time, multiple online applications can use it without extra copies and downloads.

## QUICK START

### Preface

This tutorial explains how developers can use and produce Cloud Optimized GeoTIFFs.

### Reading

The simplest way is to use GDAL’s [VSI Curl](http://gdal.org/cpl__vsi_8h.html#a4f791960f2d86713d16e99e9c0c36258) functionality. See the “How to read it with GDAL” section in the GDAL Wiki: [How to read it with GDAL](https://trac.osgeo.org/gdal/wiki/CloudOptimizedGeoTIFF#HowtoreaditwithGDAL). Most current geospatial software uses GDAL as a dependency, so adding GDAL is the fastest way to get COG reading capabilities.

On [Planet](http://planet.com/), all data is already in COG format, and there is a short guide on downloading: [download part of an image](https://www.planet.com/docs/guides/quickstart-subarea/). Most of that tutorial explains how to use the Planet API, but it also shows how GDAL Warp extracts a single working area from a large COG file.

### Creation

See also the COG page on the GDAL wiki, section [How to generate it with GDAL](https://trac.osgeo.org/gdal/wiki/CloudOptimizedGeoTIFF#HowtogenerateitwithGDAL).

```shell
$ gdal_translate in.tif out.tif -co TILED=YES -co COPY_SRC_OVERVIEWS=YES -co COMPRESS=DEFLATE
```

Or use the [rio-cogeo](https://github.com/cogeotiff/rio-cogeo) plugin:

```shell
$ rio cogeo create in.tif out.tif --cog-profile deflate
```

Many other geospatial tools should also be able to add appropriate overviews and tiles.

### Validation

Using the rio-cogeo plugin:

```shell
$ rio cogeo validate test.tif
```

## References

https://www.cogeo.org/