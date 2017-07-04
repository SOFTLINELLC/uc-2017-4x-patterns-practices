<!-- .slide: data-background="./reveal.js/img/title.png" -->

<!-- Presenter: Matt -->
# ArcGIS API 4.0 for JavaScript: Patterns and Best Practices

![qr](./images/qr.jpg)

## Short URL: bit.ly/4xpatterns

---

![Welcome](./images/welcome.gif)

### Kelly Hutchins – [@kellyhutchins](https://twitter.com/kellyhutchins)
### Matt Driscoll – [@driskull](https://twitter.com/driskull)

---

# Agenda

- 4x Goals
- 3x &#8594; 4x Migration
- 4x Significant changes
- 4x Tips & Best Practices
- Resources

---

# Goals: 4x JavaScript API

- 2D/3D Visualization
  - web maps and scenes
- Improved developer experience
  - consistency
  - predictable
  - promises
- Widgets
  - Modernize
  - Library/Framework agnostic
  - User experience
- Enhance documentation
- Improve integration with `Portal`
- Use cutting edge browser features

---

# Major Design Changes

- AMD only
- New Development patterns
- Map/View separation
- Widget View/ViewModel separation
- `Accessor` class
- Widgets
- Popups

---

# Supported Browsers (modernization)

- Chrome
- Firefox
- Edge
- Safari 7.1+
- iOS Safari
- IE11*

*IE11 WebGL is not optimized. May not perform well when opening certain scenes.

---

<!-- Presenter: Kelly -->
# 4.x Migration

![Migration](./images/migration.gif)

[Migration Guide ](https://developers.arcgis.com/javascript/latest/guide/migrating/index.html)

---

# Functionality Matrix
[![Functionality Matrix](./images/matrix.png)](https://developers.arcgis.com/javascript/latest/guide/functionality-matrix/index.html#arcgisdynamicmapservicelayer)

More to come

---

# CSS Theme
- main.css - styles for everything API.
```html
<link rel="stylesheet" href="https://js.arcgis.com/4.0/esri/css/main.css">
```
- view.css - basic map with default widgets
```html
<link rel="stylesheet" href="https://js.arcgis.com/4.0/esri/css/view.css">
```

---

# New SDK
[![SDK](./images/sdk_tour.png)](https://developers.arcgis.com/javascript/)

---


<!-- Presenter: Matt -->
# 4.x Significant changes

![more](./images/more.gif)

---

<!-- Presenter: Matt -->

# Accessor

- abstract class
- facilitates the access to instance properties
- mechanism to watch for property changes
- inherited by many Esri classes
- Provides a common developer experience

> Inheritance: View &#8594; Accessor

[SDK example](https://developers.arcgis.com/javascript/latest/api-reference/esri-views-MapView.html)

---

# Accessor Benefits

- No need for events
  - Stateful properties to watch
- Leaner SDK: we doc only the properties, the rest is convention
- Common foundation for our API
- AutoCasting

---

# Accessor: Autocasting

```js
var symbol = new SimpleMarkerSymbol({
  style: "diamond",
  color: [255, 128, 45],  // No need to write new Color()
  outline: {              // No need for new SimpleLineSymbol()
    style: "dash-dot",
    color: [255, 128, 45] // Again, no need for new Color()
  }
});
```

VS

```js
var symbol = new SimpleMarkerSymbol({
  style: "diamond",
  color: new Color([255, 128, 45]),
  outline: new SimpleLineSymbol({
    style: "dash-dot",
    color: new Color([255, 128, 45])
  })
});
```

---

# Accessor: Getting properties

```js
var basemap = map.basemap;
```

```js
var basemapTitle = map.get("basemap.title");
```

## No need for...

```js
var basemapTitle = map.basemap && map.basemap.title;
```

---

# Accessor: Setting properties

```js
 view.center = [ -100, 40 ];
 view.zoom = 6;
 map.basemap = 'oceans';
```

---

# Accessor: Watching properties

```js
var handle = map.watch('basemap.title',
function(newValue, oldValue, property, object) {
  console.log(newValue, oldValue, property, object);
});
```

---

# Accessor: Property Demo

[Demo](./demo/accessor-properties/)

---

# watchUtils

- utilities and convenience functions
- for watching `Accessor` properties

```js
watchUtils.init(accessorClass.property,
function(newValue, oldValue, propertyName, target){
  console.log(newValue, oldValue, propertyName, target);
});
```

[SDK](https://developers.arcgis.com/javascript/latest/api-reference/esri-core-watchUtils.html)

---

# Collections

- Like a "super" array.
- Supports JS Array functions `forEach()` etc
- stores an array of items of the same type
- provides useful utility methods for working with items
  - filter()
  - find()
  - reduce()

examples: map.layers, popup.actions, etc.

---

# Collections: Event

- change event
- fires each time an item is
  - added
  - moved
  - removed

---

# Promises

- Classes may be a Promise
 - Load resources
 - Asychronously initialized `Layer`, `WebMap`, `WebScene`, `View`

---

# Promises: What are they?

- handle asynchronous operations
- future value returned
- 3 states
  - pending
  - resolved
  - rejected
- Methods
  - then

---

# Promises: comparison

## 3.x

```javascript
// map loaded
if (map.loaded) {
  init();
} else {
  on.once(map, 'load', init);
}

```

## 4.x
```javascript
view.then(init);
```

---

# Promises: then()

- commonly used with the `.then()` method
 - allows you to define the callback and errback functions

```js
someAsyncFunction().then(function(resolvedVal){
  // This is called when the promise resolves
  console.log(resolvedVal);
}, function(error){
  // This function is called when the promise is rejected
  console.error(error);
});
```

---

# Promises: Rejected

When a promise is rejected, it should be handled in an errback function.

---

# Promises: Why?

- Better than an event listener
- Access the result of an asynchronous process directly after it completes
- If you initialize an event listener after an event has occurred then the listener will never fire

---

# Promises: Learn more

[Working with promises](https://developers.arcgis.com/javascript/latest/guide/working-with-promises/index.html)

---

# Loadable

- An extension of promise
- Starts async process once load() is called
- Used on layers
  - Layers shouldn't start async process until needed

---

# Loadable: Layer

Example: `layer.load()` once layer is added to a view. Otherwise, don't start loading

[FeatureLayer.load()](https://developers.arcgis.com/javascript/latest/api-reference/esri-layers-FeatureLayer.html#load)

---

# LayerView

- Gives info about a layer's rendering on a view
  - features
  - after it has been added to either a `MapView` or a `SceneView`.
- Can override properties of the layer
  - visibility, etc.
- Views have a `Collection of LayerViews`

---

# FeatureLayer

- Can be created from
  - Map service
  - feature service
  - AGOL/Portal items
  - client side graphics
- Supports Map/Scene views
- Client `definitionExpression` support. [SDK](https://developers.arcgis.com/javascript/latest/api-reference/esri-layers-FeatureLayer.html#definitionExpression)
- Not currently supported
  - adding geometries
  - deleting geometries
  - editing geometries
- Future editing support

---

# FeatureLayer: Portal Item

```js
// points to a hosted Feature Layer in ArcGIS Online
var fl = new FeatureLayer({
  portalItem: {  // autocasts as esri/portal/PortalItem
    id: "8444e275037549c1acab02d2626daaee"
  }
});
map.add(fl);  // adds the layer to the map
```

---

# FeatureLayer: Graphics

```js
lyr = new FeatureLayer({
   fields: [
   {
     name: "ObjectID",
     alias: "ObjectID",
     type: "oid"
   },...],
   geometryType: "point",
   spatialReference: { wkid: 4326 }
   source: graphics  //  an array of graphics
});
```

---

# GroupLayer

- Organize layers into common layer
- listMode - how the layer should display in ToC
  - show
  - hide
  - hide-children
- visibilityMode - Visibility of the children layers
  - independent
  - inherited
  - exclusive

  [SDK](https://developers.arcgis.com/javascript/latest/api-reference/esri-layers-GroupLayer.html)

---

## GroupLayer: Demos

- [Webscene Demo](http://jsapi.maps.arcgis.com/home/webscene/viewer.html?webscene=75a84d15aa254e47ab44298c2ff18f19)
- [GroupLayer Demo](./demo/grouplayer/index.html)

---

# Widgets

- Esri Dijits are now called Widgets
- Simplified getting, setting, and watching widgets to be more simplistic and consistent
- Widgets extend a form of `Accessor` + `_WidgetBase`

![Widgets](./images/widgets.png)

---

# Widgets: Enhancements

- Combined Attribution and logo widgets into new [Attribution widget](https://developers.arcgis.com/javascript/latest/sample-code/webmap-basic/live/index.html)
- Divided the Locate widget into updated Locate widget and new [Track widget](https://developers.arcgis.com/javascript/latest/sample-code/widgets-track/index.html)
- Updated design to match Esri "Calcite" styling
  - More modern looking
- Added 3D specific widgets
  - [Compass](https://developers.arcgis.com/javascript/latest/api-reference/esri-widgets-Compass.html)
  - [NavigationToggle](https://developers.arcgis.com/javascript/latest/api-reference/esri-widgets-NavigationToggle.html)
- Redesigned Popups

[Demo](http://localhost/git/4.0master/test-apps/widgets/ui-corners/ui-corners.html)

---

# Widgets: View Models

- New architecture
- Logic of the widget separated from the representation
- View implementations made in `Accessor`
- Views' source code available in the SDK
- View's can be rewritten in any framework
- ViewModels can be combined to create Frankenwidgets

---

# Widgets: SASS

- Widgets in API are now styled using [SASS](http://sass-lang.com/)
- CSS Preprocessor
- Variables, Mixins
- Allow for easier customization

![SASS](./images/sass.png)

---

# Widgets: Theming

- [theming with CSS](./demo/theming/index.html)
- [theming with SASS](http://localhost/git/4.0master/test-apps/widgets/ui-corners/ui-corners.html)

---

# Popup Widget

- [Popup Demo](./demo/widgets/popup/custom-actions.html)
- [Popup Dock Positions](https://developers.arcgis.com/javascript/latest/sample-code/popup-docking-position/index.html)
- [Popup Pagination + Menu](http://localhost/git/4.0master/test-apps/widgets/popup/2d-async-features.html)

![popup](./images/popup.png)

---

# PopupTemplate

- [PopupTemplate](https://developers.arcgis.com/javascript/latest/api-reference/esri-PopupTemplate.html) redesigned
- Can have a title and a content
- custom action buttons

```js
var template = new PopupTemplate({
  title: "My Title",
  content: "My Content",
  fieldInfos: [...],
  actions: Collection([...]),
  overwriteActions: false
});
```

---

# PopupTemplate: Content

- Content can be a string or an array of objects
- Content array has different types
  - media
  - text
  - attachments
  - fields

```js
var template = new PopupTemplate({
  content: [{
    type: "text",
    text: "There are {Point_Count} trees"
  },
  {
    type: "media",
    mediaInfos: [...]
  },
  {
    type: "attachments"
  }]
});
```

---

# PopupTemplate: Actions

- Added support for [`actions`](https://developers.arcgis.com/javascript/latest/api-reference/esri-PopupTemplate.html#actions)
- `actions` are custom buttons to do something app specific
- Can have an icon and a text
- Listen to `trigger-action` event to call your own function
- By default, just "zoom-to" is in `actions`

---

# Widgets: Current

- Out of the box widgets at 4.0:
 - Zoom
 - Attribution
 - Compass
 - Home
 - Locate
 - Search
 - Legend
 - Popup

---

# Widgets: future

- Still many widgets to port!
- Adding functionality
- Improving UX/UI
- Improving Accessibility

---

<!-- Presenter: Kelly -->
# Significant changes: Map

- Map.graphics is a collection, not a layer
- Map can be displayed in 2d or 3d

---

# Basemaps
- Full fledged class  `esri/Basemap`
- Basemap
  - `baseLayers`
  - `referenceLayers`
- can be set with
  - [string for esri's basemap](demo/basemap/2d.html)
  - Custom [Basemap instance](demo/basemap/custom-arctic.html)
  - [Toggle](demo/map/togglebasemaps.html) Basemaps
---

# Significant changes: View
![View](./images/api-diagram-0b.png)


---

# Map & View separation

Map (data) and view (presentation) are broken apart.

## 3.x
```javascript
var map = new Map( "mapDiv", {
  basemap: "topo"
});
```

## 4.x
```javascript
var myMap = new Map({
  basemap: "streets"
});
var view = new MapView({
  map: myMap,  // References a Map instance
  container: "viewDiv"  // References the ID of a DOM element
});
```

---

# Map/View separation: Multiple views

```javascript
require([
  "esri/Map",
  "esri/views/MapView",
  "esri/views/SceneView"
], function(Map, MapView, SceneView) {

  var myMap = new Map({
    basemap: 'streets'
  });

  var 2dView = new MapView({
    map: myMap
  });

  var 3dView = new SceneView({
    map: myMap
  });
});
```
[Demo](https://developers.arcgis.com/javascript/latest/sample-code/views-synchronize/index.html)

---

# Map/View separation: Camera
- Set the view to a target which can include
  - longitude, latitude pair
  - Graphic or geometry
  - Viewpoint
  - Camera

  ```js
  var cam = new Camera({
    position: new Point({
      x: -100.23, // lon
      y: 65,      // lat
      z: 10000,   // elevation in meters
    }),
    heading: 180, // facing due south
    tilt: 45      // bird's eye view
  });

  view.goTo(cam);
  ```

---

# Map/View separation: Navigation
- Animate the view to a target which can include
  - longitude, latitude pair
  - Graphic or geometry
  - Viewpoint

  ```js
  var pt = new Point({
    latitude: 49,
    longitude: -126  
  });
  view.goTo(pt);
  ```
[Demo](demo/map/goto/index.html)

---

# Significant changes: Webmap + Webscene
- Full Web Scene support
- Partial Web Map support
  - Non supported layers won't show (UnsupportedLayer)
  - Full support coming late 2016

---

# Webmap

```js
var webmap = new WebMap({
  portalItem: {
    id: 'e691172598f04ea8881cd2a4adaa45ba'
  }
});
```
[demo](https://developers.arcgis.com/javascript/latest/sample-code/webmap-basic/live/index.html)


---

<!-- Briefly -->
# WebScene

```js
var webscene = new WebScene({
  portalItem: {
    id: '19dcff93eeb64f208d09d328656dd492'
  }
});
```

---

# WebScene - Slides
- Created with the ArcGIS Online Scene Viewer
- Store layers visibility, camera, environment

```js
var slides = scene.presentation.slides;

// create a clickable thumbnails
slides.forEach(function(slide) {
  var thumb = new Slide({
    slide: slide
  });
  thumb.on('click', function() {
    // apply the slide on the view
    slide.applyTo(view);
  });
  slidesDiv.appendChild(thumb.domNode);
});

```

[demo](demo/platform/webscene.html)

---

<!-- Briefly -->
# Portal changes

- [redesigned API](https://developers.arcgis.com/javascript/latest/api-reference/esri-portal-Portal.html)
- Load layers, web map, web scenes
- query items, users, groups

---

```js
var portal = new Portal();

// Setting authMode to immediate signs the user in once loaded
portal.authMode = 'immediate';

// Once loaded, user is signed in
portal.load()
  .then(function() {
    // Create query parameters for the portal search
    var queryParams = new PortalQueryParams({
      query: 'owner:' + portal.user.username,
      sortField: 'numViews',
      sortOrder: 'desc',
      num: 20
    });

    // Query the items based on the queryParams created from portal above
    portal.queryItems(queryParams).then(createGallery);
  });
```

[demo](https://developers.arcgis.com/javascript/latest/sample-code/identity-oauth-basic/live/index.html)


---

# Layer from Portal Item
```js
var promise = Layer.fromPortalItem({
  portalItem: {
    id: '8444e275037549c1acab02d2626daaee',
    portal: {
      url: 'https://myorg.maps.argis.com'
    }
  }
})
.then(function(layer) {
  // Adds the layer to the map once it loads
  map.add(layer);
})
.otherwise(function(error) {
  //handle the error
});
```

[demo](demo/platform/webmap.html)

---

<!-- Presenter: Matt -->
# 4.x Best practices & tips

![changes](./images/changes.gif)

---

# Accessor

- Get comfortable with Accessor
- Accessor.get("my.property.here");

---

# Deprecated things

- Modernize your code
- no need for some things
  - lang.hitch
    - use `function(){}.bind(this)`
  - ie11+
  - border containers, form widgets
    - Use css flexbox & native input elements
  - dojo/base/array
    - Use native array functions.

---

# Renamed constant string values

## Kebab-case instead of camelCase

- `simplemarkersymbol` &#8594; `simple-marker-symbol`
- `picturemarkersymbol` &#8594; `picture-marker-symbol`
- `simplelinesymbo`l &#8594; `simple-line-symbol`

---

<!-- Presenter: Kelly -->
# Responsive General changes
Easier to build responsive apps
``` js
view.watch("widthBreakpoint", function(newVal){
  if (newVal === "xsmall"){
    // clear the view's default UI components if
    // app is used on a mobile device
    view.ui.components = [];  
  }
});
```
[Width Breakpoints](https://developers.arcgis.com/javascript/latest/api-reference/esri-views-View.html#widthBreakpoint)

[Demo](https://developers.arcgis.com/javascript/latest/sample-code/frameworks-bootstrap/index.html)

---

---

# View padding
- Offset view
  - center and extent work off subset
[![View Padding](./images/viewpadding.png)](https://developers.arcgis.com/javascript/latest/sample-code/view-padding/index.html)


---

# UI Components
![UI Components](./images/uicomponents.png)

---

# Add Components
``` js
view.ui.add(search, "top-right");
view.ui.add(logo, "bottom-right");
```

---

# Move Components
``` js
view.ui.move(logo, "bottom-left");
view.ui.remove(search);
```
[Demo](./demo/view/legend/index.html)

---

<!-- Presenter: Kelly + Matt -->
# Additional Resources

<!-- - Geonet/support/rene/github/sass (Kelly) -->
- [Documentation - 4.0 beta](https://developers.arcgis.com/javascript/beta/)
- [4x What's new](https://developers.arcgis.com/javascript/latest/guide/whats-new/index.html)
- [4x FAQ](https://developers.arcgis.com/javascript/latest/guide/faq/index.html)

---
# JSAPI Resources
- TypeScript definition files
- Bower
- JSHint

[esriurl.com/resources](http://esriurl.com/resources)

---

# Geonet
[![Geonet](./images/geonet.png)](https://geonet.esri.com/community/developers/web-developers/arcgis-api-for-javascript/tags#/?recursive=false&tags=4.0)


---

# Blogs
- [ArcGIS Blog](https://blogs.esri.com/esri/arcgis/tag/javascript/)
- [Odoe.net](http://odoe.net/blog/)


---

# Get The Code

## [bit.ly/4xpatterns](http://bit.ly/4xpatterns)

![code](./images/code.gif)

---

# Please take our survey

## Your feedback allows us to help maintain high standards and to help presenters

![Rate us](./images/rate-us.png)

---

# Questions?

![questions](./images/questions.gif)

---

![Thanks](./images/thanks.gif)

---

<!-- .slide: data-background="./reveal.js/img/end.png" -->
