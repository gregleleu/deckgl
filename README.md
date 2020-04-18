
<!-- README.md is generated from README.Rmd. Please edit that file -->
An R Interface to deck.gl
=========================

[![CRAN\_Status\_Badge](https://www.r-pkg.org/badges/version/deckgl)](https://cran.r-project.org/package=deckgl) [![Travis-CI Build Status](https://travis-ci.org/crazycapivara/deckgl.svg?branch=master)](https://travis-ci.org/crazycapivara/deckgl) [![Project Status: Active – The project has reached a stable, usable state and is being actively developed.](https://www.repostatus.org/badges/latest/active.svg)](https://www.repostatus.org/#active) [![deck.gl Version](https://img.shields.io/badge/deck.gl-v8.1.0-blue.svg)](https://github.com/uber/deck.gl/releases/tag/v8.1.0)

Deckgl for R makes the open-source JavaScript library [deck.gl](https://deck.gl/) available within R via the [htmlwidgets](https://www.htmlwidgets.org/) package.

Installation
------------

``` r
install.packages("deckgl")
```

You can install the latest version of deckgl from github with:

``` r
# install.packages("remotes")
remotes::install_github("crazycapivara/deckgl")
```

Usage
-----

Create a *deckgl* widget:

``` r
deckgl()
```

Add a basemap:

``` r
deckgl() %>%
  add_basemap()
```

Add any kind of layers:

``` r
# Grid layer example
data("sf_bike_parking")

props <- list(
  extruded = TRUE,
  cellSize = 200,
  elevationScale = 4,
  getPosition = ~lng + lat,
  tooltip = "Count: {{count}}"
)

deckgl(zoom = 11, pitch = 45) %>%
  add_basemap() %>%
  add_grid_layer(data = sf_bike_parking, properties = props)
```

Layers
------

Due to the generic function `add_layer` any kind of layer defined in the [deck.gl API Reference](https://deck.gl/#/documentation/deckgl-api-reference) is supported. The *layer type* is chosen via the `class_name` parameter, e. g. `ScatterplotLayer` or `GeoJsonLayer`. Usually you will not use the generic function but one of the `add_*_layer` shortcut functions instead:

``` r
# Generic function
deckgl() %>%
  add_layer("ArcLayer", id, data, properties)

# Shortcut function
deckgl() %>%
  add_arc_layer(id, data, properties)
```

Data
----

The `data` parameter can either be an *url* to fetch data from or a *data object*. In most cases you will pass an object of type `data.frame` to the layers. To define *data accessors* (functions that are used by *deck.gl* to access the properties of the data object), you use the formula syntax in R. For example, if the coordinates are contained in the columns `lng` and `lat` you set your `getPosition` prop like this:

``` r
props <- list(
  getPosition: ~lng + lat
  # ...
)
```

### Data objects of class *sf*

If you pass an object of class [sf](https://github.com/r-spatial/sf) to the layers you just need to tell the corrosponding prop that fetches the geometry to use the *geometry* column. In the case of the `PolygonLayer` this would be as follows:

``` r
props <- list(
  getPolygon = ~geometry
  # ...
)
```

Layer Props
-----------

The `properties` parameter is a *named list* with names corresponding to the *properties* defined in the [deck.gl API Reference](https://deck.gl/#/documentation/deckgl-api-reference) for the given *layer type* (class). For the example above see the [GridLayer API Reference](https://deck.gl/#/documentation/deckgl-api-reference/layers/grid-layer).

It is also possible to pass properties as named arguments via the `...` parameter. They are appended to the properties list. Same properties will be overwritten.

See the [deck.gl Layer Catalog](https://github.com/visgl/deck.gl/tree/master/docs/layers#deckgl-layer-catalog-overview) for a full list of available layers and their properties.

### Camels or Snakes

While *deck.gl* uses *camelCased* parameters according to the JavaScript standard, the R style conventions usually prefer the *snake\_case* standard. Therefore, all parameters that are passed to *deck.gl* can use *snake\_cake*, e. g. `getPosition` in *deck.gl* can be passed as `get_position`.

### Data Accessors

Use the *formula* syntax in R if a property is a *data accessor*:

``` r
props <- list(
  getPosition = ~lng + lat # js: d => [d.lng, d.lat]
  getFillColor = ~color # js: d => d.color
  # ...
)
```

The example above assumes that your data contains the columns `lng`, `lat` and `color`. It is also possible to pass JavaScript code by using the `JS` function in R:

``` r
props <- list(
  getColor = JS("d => d.capital ? [140, 10, 10] : [60, 10, 10]")
  # ...
)
```

### Colors

*Deck.gl* uses rgba arrays (`[r, g, b, a]`) to represent colors. Nevertheless you can pass hex color codes to all color arguments of the `add_*_layer` functions. They are automatically converted to the required rgba arrays.

Tooltips
--------

The tooltip for a layer can be set via the `tooltip` parameter. You can either pass a single template string or a list with the following properties (see also `use_tooltip`):

-   `html`: A template string that will be set as the `innerHTML` of the tooltip.
-   `style`: A `cssText` string that will modefiy the default style of the tooltip.

### Tooltip template Syntax

The tooltip string is a *mustache* template in which variable names are identified by the double curly brackets (*mustache* tags) that surround them. The variable names available to the template are given by deck.gl’s `pickingInfo.object` and vary by layer.

See [mustache](https://github.com/janl/mustache.js) for a complete syntax overwiew.

``` r
data("bart_segments")

props <- list(
  getWidth = 12,
  getSourcePosition = ~from_lng + from_lat,
  getTargetPosition = ~to_lng + to_lat,
  getSourceColor = "yellow",
  getTargetColor = "orange",
  tooltip = use_tooltip(
    html = "{{from_name}} to {{to_name}}",
    style = "background: steelBlue; border-radius: 5px;"
  )
)

deckgl(zoom = 9.5, pitch = 35) %>%
  add_arc_layer(data = bart_segments, properties = props) %>%
  add_basemap()
```

Controls
--------

*Controls* are elements that are displayed as overlays on top of the map / deck. Usually you can set the *position* and the *style* of the control. The most basic control is a simple text box:

``` r
deckgl() %>%
  add_basemap() %>%
  add_control(
    html = "Plain Base Map",
    pos = "top-right",
    style = "background: steelblue; color: white"
  )
```

### Legends

...

Basemaps
--------

Using `add_basemap` adds a [carto Basemap](https://carto.com/developers/carto-vl/reference/#cartobasemaps) by default. To use basemaps from [mapbox](https://www.mapbox.com/maps/) it is recommended that you set your API access token in an environment variable called `MAPBOX_API_TOKEN`:

``` r
# If not set globally
#Sys.setenv(MAPBOX_API_TOKEN = "xyz")

deckgl() %>%
  add_mapbox_basemap("mapbox://styles/mapbox/light-v9")
```

Run Examples
------------

You can run the [API Examples](https://github.com/crazycapivara/deckgl/tree/master/inst/examples/deckgl-api-reference) from the `add_*_layer` functions with `example(add_*_layer)`. For the `IconLayer` it looks like this:

``` r
example(add_icon_layer)
```

Development
-----------

The JavaScript library of the *deckgl widget* uses [webpack](https://webpack.js.org/) as module bundler. Therefore, you need [node.js](https://nodejs.org) to build the *widget* library.

First of all you need to clone the repository:

``` bash
git clone https://github.com/crazycapivara/deckgl.git
```

Then you can build the library from inside the `javascript` folder like this:

``` bash
cd deckgl/javascript

npm run build
```

All JavaScript code is located in the `src` folder and test components go to the `src/test-components` folder.

To test the library just run the following command. This will spin up the [webpack-dev-server](https://github.com/webpack/webpack-dev-server):

``` bash
npm run start
```

Concept
-------

*Deckgl for R* stays as close as possible to the JavaScript API so that usually all parameters of its JavaScript pendants are supported. Therefore, you need to check the [deck.gl API Reference](https://deck.gl/#/documentation/deckgl-api-reference) of the JavaScript framework to get information about the parameters you can pass to the R-functions mostly as *named lists* or *named arguments* (`...` parameter). Use the `JS` function if you need to pass any kind of JavaScript code. *camelCased* parameters in *deck.gl* can be passed as *snake\_cased* parameters in R.

[GridLayer](https://deck.gl/#/documentation/deckgl-api-reference/layers/grid-layer) API Example:

``` javascript
// JavaScript code

const layer = new GridLayer({
  id: "grid-layer",
  data: data,
  extruded: true,
  cellSize: 200,
  elevationScale: 4,
  getPosition: d => [d.lng, d.lat]
});
```

``` r
# Corresponding R code

deck <- deckgl() %>%
  add_grid_layer(
    id = "grid-layer",
    data = data,
    extruded = TRUE,
    cell_size = 200,
    elevation_scale = 4,
    get_position = ~lng + lat
  )
```

In this example all properties are passed as named arguments.

Documentation
-------------

-   [deckgl for R](https://crazycapivara.github.io/deckgl/) (pkgdown-site)
-   [example scripts](https://github.com/crazycapivara/deckgl/tree/master/inst/examples)
-   [deckgl-api-reference](https://deck.gl/#/documentation/deckgl-api-reference) (JavaScript framework)

Notes
-----

-   It is a known issue that the *deckgl widget* might not be visible in the viewer pane of RStudio. Just open it in your browser by clicking *Show in new window* and everything will be fine.
-   The [documentation](https://crazycapivara.github.io/deckgl/) is work in progress. See also the [examples](https://github.com/crazycapivara/deckgl/tree/master/inst/examples) as a good starting point.
