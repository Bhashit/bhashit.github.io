---
layout: post
title:  Understanding open layer maps
date:   2014-06-22
comments: true
---


## Basics

The open layers API has two basic concepts: Map and Layer. A Map stores information about the default projections, units and other details of the map. Inside the map, the data is displayed via Layers. A Layer is a data source; information about how open layers should request data and display it.

OpenLayers allows putting a Map inside any block level HTML element.

The most important APIs relevant to the basic maps construction are:

1. [OpenLayers Map javascript API][open_layer_map_api]
2. [OpenLayer Google Map Layer javascript API][open_layer_gmap_api]


```html
<html>
<head>
  <title>OpenLayers Example</title>
    <script src="http://openlayers.org/api/OpenLayers.js"></script>
    <script>
      window.onload = function() {
        var map = new OpenLayers.Map('map');
      }
    </script>
    </head>
    <body>
      <div style="width:100%; height:100%" id="map"></div>
    </body>
</html>

```

The `OpenLayers.Map` constructor requires one argument. This argument must be an HTML element or the id of an HTML element. The `OpenLayers.Map` constructor creates the map viewer.

### Adding the map layer to the map viewer

Now, we add the actual map layer to the map viwer created above. Since we are going to use Google maps, we have to make the changes accordingly. To begin with, we first need to deal with the concept of [map projections][map_projection]. Different maps services may use different kinds of map projections. Google maps uses a projection called [Mercator projection][google_maps_mercator]. Mercator projection is identified by [ESPG:3857][espg_3857]. So, we have to tell our map viewer what kind of map projections it's going to use. The [default projection][open_layer_default_projection] used by OpenLayers is EPSG:4326.

```javascript
var map = new OpenLayers.Map('map', {
  projection: 'EPSG:3857'
};
```

Now, we add the actual map layers.

```javascript
var satelliteLayer = new OpenLayers.Layer.Google(
  "Google Satellite", // name of the layer
  { type: google.maps.MapTypeId.SATELLITE, numZoomLevels: 22 }
);


var center: new OpenLayers.LonLat(2.294694, 48.858093).transform(
  new OpenLayers.Projection("EPSG:4326"), map.getProjectionObject());
var zoomLevel = 17;
map.addLayer(satelliteLayer);
map.setCenter(center, zoomLevel);

```

Note that we need to transform the coordinate to the required projection. The `numZoomLevels` are set to 22. The default value for them is 16. Different map layers may support different zoom levels. The max zoom value for the google's satellite view is 22.

This is all we need to do for a very basic example.

## Adding multiple layers using google maps

Adding multiple layers is as simple as just creating multiple `OpenLayers.Layer.Google` instances and adding them all the `map`.

```javascript

var map = new OpenLayers.Map('map', {
  projection: 'EPSG:3857'
});

var satelliteLayer = new OpenLayers.Layer.Google(
  "Google Satellite",
  // The google maps satellite layer support 22 zoom levels.
  {type: google.maps.MapTypeId.SATELLITE, numZoomLevels: 22}
);

var roadMapLayer = new OpenLayers.Layer.Google(
  "Google Roadmap", // the default
  {type: google.maps.MapTypeId.ROADMAP, numZoomLevels: 20}
);

var terrainLayer = new OpenLayers.Layer.Google(
  "Google Terrain",
  {type: google.maps.MapTypeId.TERRAIN}
);

var hybridLayer = new OpenLayers.Layer.Google(
  "Google Hybrid",
  {type: google.maps.MapTypeId.HYBRID, numZoomLevels: 20}
);

var allLayers = [satelliteLayer, roadMapLayer, terrainLayer, hybridLayer];

var zoom = 17;
map.addLayers(allLayers);
map.setCenter(new OpenLayers.LonLat(2.294694, 48.858093).transform(
  new OpenLayers.Projection("EPSG:4326"),
  map.getProjectionObject()
), zoom);

// Add the control that allows switching between different layers.
map.addControl(new OpenLayers.Control.LayerSwitcher());

```

## More about layers in OpenLayers

OpenLayers has two different types of layers: Base layers and non-base layers.

### Base Layers

Base layers are mutually exclusive layers. Only one base layer can be enabled at a time. The currently active base layer determines the available projection and zoom levels available to the map. If a given base layer doesn't specify its own projection, the projection specified by the map object is used.

Whether a layer is a base layer or not can be specified using the [`isBaseLayer`][open_layer_is_base_layer] property of the `Layer` object. Most layers have this property set to `true` by default. Base layers are always displayed below the overlay layers.

Google maps layers are raster layers. Raster layers provide the imagery for maps. These layers are designed to be used only as a base layer.

### Non-base layers

Non-base layers are also called overlays. Multiple overlays may be enabled at the same time. These layers do not control the zoom levels of the map, but can be enabled or disabled at certain scales by min/max scale/resolution  parameters so that they are only enabled at certain level.

Overlay layers generally have their source data in a format other than imagery. Overlay layers include things like markers and vectors.

## Adding markers

Adding markers to OpenLayers is not as straightforward as with Google or Bing maps. OpenLayer markers are a combination of [`OpenLayers.LonLat`][open_layer_lonlat] and [`OpenLayers.Icon`][open_layer_icon]. Markers are added to a special layer, which is an instance of [`OpenLayers.Layer.Markers`][open_layer_markers].

```javascript
var markersLayer = new OpenLayers.Layer.Markers( "Markers" );
// The size of the icon
var size = new OpenLayers.Size(21,25);
// The amount of offset for the icon from its anchor point.
var offset = new OpenLayers.Pixel(-(size.w/2), -size.h);
var icon = new OpenLayers.Icon('http://www.openlayers.org/dev/img/marker.png', size, offset);

var marker = new OpenLayers.Marker(center, icon);
staticMarkersLayer.addMarker(new OpenLayers.Marker(center, icon));
staticMarkersLayer.addMarker(new OpenLayers.Marker(LatLong(map, 48.857, 2.294), icon.clone()));
```

Note that while adding the second icon, we are cloning the icon instance instead of reusing it. This is required by the OpenLayers. According to the OpenLayers docs, icon instances should not be shared between markers.

There are several things that you can do with markers, like setting the opacity of the layers and adding click events and so forth. The markers don't allow themselves to be dragged by default. The OpenLayers documentations doesn't have any details on draggable markers. If we want to make the markers draggable, we can create them ourselves using [`vectors`][vectors]. More on that later.

## Adding Popups

[map_projection]: https://en.wikipedia.org/wiki/Map_projection
[google_maps_mercator]: https://groups.google.com/forum/#!msg/Google-Maps-API/gZytmchfZB4/5vYheY6xIoIJ
[espg_3857]: https://wiki.openstreetmap.org/wiki/EPSG:3857
[open_layer_map_api]:http://dev.openlayers.org/releases/OpenLayers-2.13.1/doc/apidocs/files/OpenLayers/Map-js.html
[open_layer_gmap_api]: http://dev.openlayers.org/releases/OpenLayers-2.13.1/doc/apidocs/files/OpenLayers/Layer/Google-js.html
[open_layer_default_projection]: http://dev.openlayers.org/releases/OpenLayers-2.13.1/doc/apidocs/files/OpenLayers/Map-js.html#OpenLayers.Map.projection
[open_layer_is_base_layer]: http://dev.openlayers.org/releases/OpenLayers-2.13.1/doc/apidocs/files/OpenLayers/Layer-js.html#OpenLayers.Layer.isBaseLayer
[open_layer_lonlat]: http://dev.openlayers.org/apidocs/files/OpenLayers/BaseTypes/LonLat-js.html#OpenLayers.LonLat
[open_layer_icon]: http://dev.openlayers.org/apidocs/files/OpenLayers/Icon-js.html#OpenLayers.Icon
[open_layer_markers]: http://dev.openlayers.org/apidocs/files/OpenLayers/Layer/Markers-js.html#OpenLayers.Layer.Markers
[open_layers_vectors]: http://dev.openlayers.org/apidocs/files/OpenLayers/Layer/Vector-js.html
