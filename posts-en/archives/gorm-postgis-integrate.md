---
id: aYI
title: Gorm + Postgis Accessing Spatial Geographic Data
pubDate: 2019-12-10T12:24:46.000Z
isDraft: true
tags:
  - openlayers
Categories:
  - 笔记
---

I’d been using Spring Boot to build small projects. It’s convenient, but always felt a bit hazy. Recently I spent some time learning **golang**. Although some features are a bit cumbersome to implement, I can read library source code to solve my own needs, and that feels pretty good during the learning process.

### Requirements

-   [ ] PostgreSQL as data source.
-   [ ] Store data in PostGIS and publish via GeoServer.
-   [ ] Frontend can add/delete/update/query points, with database and frontend kept in sync.

### Problems encountered

#### 1. Choosing the type for Geom in the Model

Because I needed to convert between WKT and WKB later, I did a quick search and picked the [github.com/paulmach/orb](github.com/paulmach/orb) library, which provides WKT and WKB decoding.

Initially I used `orb.Point` directly, and naturally it failed badly.

PostGIS requires wrapping `geom` field reads/writes with functions, for example:

```sql
st_geomfromewkt()
st_geomfromewkb()
```

Since using the type directly didn’t work (very likely I was using it wrong, because in theory `orb.Point` has implemented `Scanner()` and `Value()`), my second idea was to skip gorm’s abstraction entirely and just use pure SQL inserts.

That simplified things. The statement looked like this:

```go
err = db.Exec(fmt.Sprintf("update workareas set geom = ST_GeomFromEWKT('SRID=4326;%s') where id = %d",wkt.MarshalString(*point),wa.ID)).Error
```

Because I still wanted to use gorm’s model features and soft delete, I first saved the non‑`geom` fields, then ran an update. This worked fine before linking the table to GeoServer, but as soon as I published the layer, the table’s `geom` field was automatically set to `Not Null`, which was really annoying. So this approach was no longer viable.

The main issue now was that using gorm’s `Create()` method, I couldn’t bypass the PostGIS functions that are usually used for storing the geom field. I tried a few approaches without success. So I connected to the database directly and experimented. I found that when storing as WKT, you can omit the external function and it still works, like this:

```sql
update workareas set geom = 'SRID=4326;POINT(111 55)' where id = 17
```

This syntax is equivalent to:

```sql
update workareas set geom = st_geomfromewkt('SRID=4326;POINT(111 55)') where id = 17
```

That made things easier. The final type definition for the sample point looks like this:

```go
type Model struct {
	ID        uint       `gorm:"primary_key" json:"id"`
	CreatedAt time.Time  `json:"created_at"`
	UpdatedAt time.Time  `json:"updated_at"`
	DeletedAt *time.Time `sql:"index" json:"deleted_at"`
}

type Workarea struct {
	Model
	Geom string
	Name string
}
```

This way we can store it like this:

```go
point := &orb.Point{lon, lat}
wa := model.Workarea{Name: "wode", Geom: fmt.Sprintf("SRID=4326;%s", wkt.MarshalString(*point))}
err := db.Create(&wa).Error
// error handling omitted
```

Once storage is solved, update and delete are straightforward.

#### 2. Issues on the frontend

Mainly related to OpenLayers’ `draw` and `modify` events.

Getting coordinates for new or modified points:

```javascript
// Getting coordinates after a point is moved; it’s a bit cumbersome, pulled from the event object.
gModify.on(['modifyend'], function (e) {
    // The feature that was modified
    let mFeature
    // This gets all features in the source
    let features = e.features.getArray()
    // These are the coordinates after moving
    let tcr = e.target.lastPointerEvent_.coordinate
    // Since I didn’t find a way to directly get the modified feature
    // Using `Revision()` can’t reliably find it when multiple edits happen
    for (let i = 0; i < features.length; i++) {
        if (
            tcr[0] === features[i].getGeometry().getCoordinates()[0] &&
            tcr[1] === features[i].getGeometry().getCoordinates()[1]
        )
            mFeature = features[i]
    }
    // Coordinates to be stored in the database need a coordinate transformation
    // No need to use geo.transform(**) here
    // Because geo is actually a reference to the newly drawn feature
    // Directly transforming would change the newly added feature’s coordinates,
    // making it not display due to different coordinate systems
    let tcr2 = transform(tcr, new Projection({ code: 'EPSG:3857' }), new Projection({ code: 'EPSG:4326' }))
    // The method in the comments below is wrong:
    // it would cause the drawn feature to fail to display correctly because of wrong coordinates.
    // geo.transform(new Projection({code:"EPSG:4326"}),new Projection({code:"EPSG:3857"}))
    //let coords = geo.getCoordinates();
    //console.log(coords)

    console.log(mFeature.getId())
    // Get the feature id to update the database
    const id = mFeature.getId().split('.')[1]
    //console.log(id)
    // This is a fetch request
    modifyFeature('work-area', id, tcr2)
})
```

New points cannot be edited immediately:

```javascript
// Getting coordinates of the newly added point
featureDraw.on(['drawend'], function (e) {
    let feature = e.feature
    let geo = feature.getGeometry()
    console.log(geo.getCoordinates())
    // Coordinates to be stored in the database need a coordinate transformation
    // No need to use geo.transform(**) here
    // Because geo is actually a reference to the newly drawn feature
    // Directly transforming would change the newly added feature’s coordinates,
    // making it fail to display due to different coordinate systems
    let newPoint = transform(
        geo.getCoordinates(),
        new Projection({ code: 'EPSG:3857' }),
        new Projection({ code: 'EPSG:4326' })
    )
    // This is an async fetch request
    CreateFeature(layerName, newPoint)
        .then(function (r) {
            if (r.status === 404) {
                return false
            }
            console.log(`${pgUrlTable[layerName]}.${r.id}`)
            // On success, assign the id returned by the request to the feature
            // so it can be used for later edits and deletes
            feature.setId(`${pgUrlTable[layerName]}.${r.id}`)
            return true
        })
        .then((bool) => {
            if (bool) console.log('ok')
            // On failure, remove the feature that was just added
            else currentSrc.removeFeature(feature)
        })
})
```

Those are some of the small issues I ran into while building a small example of frontend dynamic point updates with gorm and PostGIS, recorded here for reference.