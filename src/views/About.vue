<template>
  <div class="about">
    <h1>This is a 2d map page</h1>
    <div id="viewDiv" />
  </div>
</template>

<style scoped>
        @import "https://js.arcgis.com/4.10/esri/css/main.css";
        @import 'https://js.arcgis.com/4.10/esri/css/main.css';
        html, body, #viewDiv {
        padding: 0;
        margin: 0;
        height: 100vh;
        width: 100vw;
      }
</style>

<script>
  import { loadModules } from "esri-loader";

export default{

  mounted(){

    const options = {
      // url: 'http://localhost:8080/arcgis_js_api/library/4.10/dojo/dojo.js'
      url: 'https://js.arcgis.com/4.10/dojo/dojo.js'
    };

    loadModules(['esri/Map', 'esri/views/MapView', 'esri/widgets/BasemapToggle',
  'esri/widgets/BasemapGallery', "esri/layers/FeatureLayer",
    //ploting a point libraries
    "esri/Graphic", "esri/geometry/Point", "esri/symbols/SimpleMarkerSymbol"], options)
    .then(([Map, MapView, BasemapToggle, BasemapGallery, FeatureLayer,
    //plotting point variables
    Graphic, EsriPoint, SimpleMarkerSymbol
    ]) => {

    // create map with the given options at a DOM node w/ id 'mapNode'
    var map = new Map({
      basemap: "topo-vector"
    });

    var view = new MapView({
      container: "viewDiv",
      map: map,
      center: [-118 ,34.09042],
      zoom: 11
    });

    // addes to the bottom right of the map a toggle between topo-vector and satellite
    var basemapToggle = new BasemapToggle({
      view: view,
      secondMap: "satellite"
    });

    view.ui.add(basemapToggle, "bottom-right");
    // end map toggle 

    // addes gallery of maps to choose from to the top right of the map
    var basemapGallery = new BasemapGallery({
        view: view,
        source: {
          portal: {
            // TODO: change to local server
            url: "http://www.arcgis.com",
            useVectorBasemaps: true  // Load vector tile basemaps
          }
        }
      });

    view.ui.add(basemapGallery, "top-right"); 
    // end map gallery choice

    // adds a layer of trails to the map
    // Define a simple renderer and symbol
    var trailheadsRenderer = {
      "type": "simple",
      "symbol": {
        "type": "picture-marker",
        "url": "https://openclipart.org/image/2400px/svg_to_png/256365/poke.png", 
        "width": 100,
        "height": 100
      },
      visualVariables: [{
        type: "size",
        field: "ELEV_FT",
        valueUnit: "feet",
        valueRepresentation: "diameter"
     }]
    }

    // Define a unique value renderer and symbols
var trailsRenderer = {
  "type": "unique-value",
  "field": "USE_BIKE",
  "uniqueValueInfos": [
    {
      "value": "Yes",
      "symbol": {
        "color": [26, 26, 26, 255],
        "width": 0.9,
        "type": "simple-line",
        "style": "dot"
      },
      "label": "Bikes"
    },
    {
      "value": "No",
      "symbol": {
        "color": [230, 0, 0, 255],
        "width": 0.9,
        "type": "simple-line",
        "style": "dot"
      },
      "label": "No Bikes"
    }
  ]
}

 // Define a class breaks renderer and symbols
  var openSpacesRenderer = {
    "type": "class-breaks",
    "field": "GIS_ACRES",
    "classBreakInfos": [
      {
        "symbol": {
          "color": [
            193,157,188,255
          ],
          "outline": {
            "width": 0
          },
          "type": "simple-fill",
          "style": "solid"
        },
        "label": "0 to 1,629",
        "minValue": 0,
        "maxValue": 1629
      },
      {
        "symbol": {
          "color": [
            203,216,162,255
          ],
          "outline": {
            "width": 0
          },
          "type": "simple-fill",
          "style": "solid"
        },
        "label": "> 1,629 to 3,754",
        "minValue": 1629,
        "maxValue": 3754
      },
      {
        "symbol": {
          "color": [
            144,198,120,255
          ],
          "outline": {
            "width": 0
          },
          "type": "simple-fill",
          "style": "solid"
        },
        "label": "> 3,754 to 11,438",
        "minValue": 3754,
        "maxValue": 11438
      }
    ]
  }

      // Trailheads point feature layer
    var featureLayer = new FeatureLayer({
      url: "https://services9.arcgis.com/wXQC4mUNRFBBF11J/ArcGIS/rest/services/griffith_park_acess/FeatureServer/0/query?where=Name%3D%27West+Trail%27&objectIds=&time=&geometry=&geometryType=esriGeometryEnvelope&inSR=&spatialRel=esriSpatialRelIntersects&resultType=none&distance=0.0&units=esriSRUnit_Meter&returnGeodetic=false&outFields=&returnHiddenFields=false&returnGeometry=true&multipatchOption=xyFootprint&maxAllowableOffset=&geometryPrecision=&outSR=&datumTransformation=&applyVCSProjection=false&returnIdsOnly=false&returnUniqueIdsOnly=false&returnCountOnly=false&returnExtentOnly=false&returnDistinctValues=false&orderByFields=&groupByFieldsForStatistics=&outStatistics=&having=&resultOffset=&resultRecordCount=&returnZ=false&returnM=false&returnExceededLimitFeatures=true&quantizationParameters=&sqlFormat=none&f=pjson&token=w_9f5GE9n_I9A1V8GITboRkk59FqgJti5tLHKiUFgTyya0sQ7Ju5hGNZEQh345i-KiZUynsfu_KY7VEshzehvr9lji9-kGqY3mHlcpxkbeUXhI3FAT4BrIFidnw6ysN2Z110bYZgR4DoyGie8iLk6TPhg1_wnVysaPMonmMWQDj_svXA-Tq7KY2Ct22CYvHqSNqp5hGARi4AZ2yanm1NwBYzbz4KdwmZZdmNswTiHV6EyEvFrcFqBntg5MC17H4c",
      renderer: trailheadsRenderer
    });

    var trailLayer = new FeatureLayer({
      url: "https://services3.arcgis.com/GVgbJbqm8hXASVYi/arcgis/rest/services/Trails/FeatureServer/0",
      //definitionExpression: "ELEV_GAIN < 250",
      renderer: trailsRenderer
    });

    var parksLayer = new FeatureLayer({
      //url: "https://services3.arcgis.com/GVgbJbqm8hXASVYi/arcgis/rest/services/Parks_and_Open_Space/FeatureServer/0"
      //renderer: openSpacesRenderer
    });

     
     map.add(parksLayer);
     map.add(trailLayer);
     map.add(featureLayer);

     //end layers to the map

    // plotting point on the map
    var point = new EsriPoint({
      longitude: -72,
      latitude: 34
    });

    var markerSymbol = new SimpleMarkerSymbol({
      color: [226, 119, 40],
      outline: {
        color: [255, 255, 255],
        width: 1
      }
    });

    var pointGraphic = new Graphic({
      geometry: point,
      symbol: markerSymbol
    });

    view.graphics.add(pointGraphic);

    //end plotting a point on the map

    })
    .catch(err => {
      // handle any script or module loading errors
      console.error(err);
      cosole.log("we got errors");
    });
  }

}
</script>

