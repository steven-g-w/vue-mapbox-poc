<template>
  <div class="hello">
    <h1>POC Map</h1>
    <button v-on:click="stelliteOn">stelliteOn</button>
    <button v-on:click="stelliteOff">stelliteOff</button>
    <button v-on:click="FlyToBrisbane">FlyToBrisbane</button>
    <button v-on:click="EaseToBrisbane">EaseToBrisbane</button>
    <button v-on:click="JumpToBrisbane">JumpToBrisbane</button>
    <button v-on:click="JumpToDC">JumpToDC</button>
    <MglMap
      :accessToken="accessToken"
      :mapStyle="mapStyle"
      @load="onMapLoaded"
      @click="hideHoverMarker"
    >
      <MglNavigationControl position="top-right" />
      <MglGeolocateControl position="top-right" />
      <MglScaleControl position="bottom-right" />
      <MglAttributionControl position="bottom-right" />

      <!-- <MglGeojsonLayer
        :sourceId="'geojson-source'"
        :source="geoJsonSource"
        layerId="myLayera"
        :layer="geojsonHeatmapLayer"
      /> -->

      <MglGeojsonLayer
        :sourceId="'geojson-source'"
        :source="geoJsonSource"
        layerId="circle"
        :layer="geojsonPrettierCircleLayer"
        @mouseenter="showCircleDetail"
        @click="circleClick"
      />

      <MglGeojsonLayer
        :sourceId="'geojson-source'"
        :source="geoJsonSource"
        layerId="text"
        :layer="geojsonTextLayer"
      />

      <MglGeojsonLayer
        :sourceId="'geojson-source'"
        :source="geoJsonSource"
        layerId="clusters"
        :layer="geojsonClusterLayer"
        @click="clusterClick"
        @mouseenter="onMouseEnter"
        @mouseleave="onMouseLeave"
      />

      <MglGeojsonLayer
        :sourceId="'geojson-source'"
        :source="geoJsonSource"
        layerId="clustersCount"
        :layer="geojsonClusterCountLayer"
        @click="clusterClick"
      />

      <MglMarker
        :coordinates="[153.03, -27.46]"
        @mouseenter="
          showHoverMarker({ content: 'ABC', coordinates: [153.03, -27.46] })
        "
      >
        <h1 style="background-color: white" slot="marker">html marker ABC</h1>
        <MglPopup>
          <div>Hello, I'm popup!</div>
        </MglPopup>
      </MglMarker>

      <MglMarker
        :coordinates="[153.04, -27.47]"
        @mouseenter="
          showHoverMarker({ content: 'EFG', coordinates: [153.04, -27.47] })
        "
      >
        <h1 style="background-color: white" slot="marker">html marker EFG</h1>
        <MglPopup>
          <div>Hello, I'm popup!</div>
        </MglPopup>
      </MglMarker>

      <MglMarker
        v-if="showingHoverMarker"
        :coordinates="showingHoverMarker.coordinates"
        :offset="[20, -15]"
      >
        <h1 style="background-color: yellow" slot="marker">
          {{ this.showingHoverMarker.content }}
        </h1>
      </MglMarker>
    </MglMap>
  </div>
</template>

<script>
import Mapbox from "mapbox-gl";
import {
  MglMap,
  MglAttributionControl,
  MglGeolocateControl,
  MglNavigationControl,
  MglScaleControl,
  MglMarker,
  MglPopup,
  MglGeojsonLayer,
} from "vue-mapbox";

export default {
  name: "HelloWorld",
  components: {
    MglMap,
    MglAttributionControl,
    MglGeolocateControl,
    MglNavigationControl,
    MglScaleControl,
    MglMarker,
    MglPopup,
    MglGeojsonLayer,
  },
  props: {},
  data() {
    return {
      accessToken:
        "pk.eyJ1IjoicnlhbmhlcmJlcnQiLCJhIjoiY2tlbndteDY0MGwzdTJ6c2F0OTc5cXlmNSJ9.LqDw6TblusSYBn475iGLMw", // your access token. Needed if you using Mapbox maps
      mapStyle: "mapbox://styles/ryanherbert/ckfnsmg3i0wsj19nx1nmc7ilt", // your map style
      showingHoverMarker: null,
      geoJsonSource: {
        type: "geojson",
        data: "/stations.geojson",
        cluster: true,
      },
      geojsonCircleLayer: {
        type: "circle",
        source: "geojson-source",
      },
      geojsonHeatmapLayer: {
        type: "heatmap",
        source: "geojson-source",
      },
      geojsonPrettierCircleLayer: {
        type: "circle",
        source: "geojson-source",
        filter: ["!", ["has", "point_count"]],
        paint: {
          "circle-color": "#11b4da",
          "circle-radius": 20,
          "circle-stroke-width": 5,
          "circle-stroke-color": "#fff",
        },
      },
      geojsonTextLayer: {
        type: "symbol",
        source: "geojson-source",
        filter: ["!", ["has", "point_count"]],
        layout: {
          "icon-image": "orange_chip_with_shadow",
          "icon-text-fit": "both",
          "icon-text-fit-padding": [3, 33, 8, 30],
          "text-anchor": "bottom",
          "text-field": "{title}",
          "text-font": ["Open Sans Bold"],
          "text-justify": "center",
          "text-size": 15,
          "text-allow-overlap": false,
          "text-letter-spacing": 0.05,
          "text-offset": [0, -1.3],
        },
        paint: {
          "text-color": "#ffffff",
        },
      },
      geojsonClusterLayer: {
        type: "circle",
        source: "geojson-source",
        filter: ["has", "point_count"],
        paint: {
          "circle-color": [
            "step",
            ["get", "point_count"],
            "#51bbd6",
            100,
            "#f1f075",
            750,
            "#f28cb1",
          ],
          "circle-radius": [
            "step",
            ["get", "point_count"],
            20,
            100,
            30,
            750,
            40,
          ],
        },
      },
      geojsonClusterCountLayer: {
        id: "cluster-count",
        type: "symbol",
        source: "geojson-source",
        filter: ["has", "point_count"],
        layout: {
          "text-field": "{point_count_abbreviated}",
          "text-font": ["DIN Offc Pro Medium", "Arial Unicode MS Bold"],
          "text-size": 12,
        },
      },
    };
  },

  created() {
    // We need to set mapbox-gl library here in order to use it in template
    this.mapbox = Mapbox;
  },
  methods: {
    onMapLoaded(event) {
      // in component
      console.log("onMapLoaded");
      this.map = event.map;
      this.map.center = [153.04, -27.47];

      this.map.loadImage("/orange_chip_icon_with_shadow.png", (error, image) => {
        if (error) throw error;
        this.map.addImage("orange_chip_with_shadow", image, {
          content: [0, 0, 57, 29],
          stretchX: [[28, 30]],
          stretchY: [[12, 14]],
        });
      });
    },
    stelliteOn() {
      this.map.setLayoutProperty("satellite", "visibility", "visible");
    },
    stelliteOff() {
      this.map.setLayoutProperty("satellite", "visibility", "none");
    },
    showHoverMarker(obj) {
      this.showingHoverMarker = obj;
    },
    showCircleDetail(event) {
      const e = event.mapboxEvent;
      const features = this.map.queryRenderedFeatures(e.point);
      const [topFeature] = features;
      console.log(topFeature.geometry.coordinates);
      this.showingHoverMarker = {
        coordinates: topFeature.geometry.coordinates,
        content: topFeature.properties.address,
      };
    },
    hideHoverMarker() {
      console.log("hideHoverMarker");
      this.showingHoverMarker = null;
    },
    FlyToBrisbane() {
      this.map.flyTo({ center: [153.03, -27.47], zoom: 13 });
    },
    EaseToBrisbane() {
      this.map.easeTo({ center: [153.03, -27.47], zoom: 13 });
    },
    JumpToBrisbane() {
      this.map.jumpTo({ center: [153.03, -27.47], zoom: 13 });
    },
    JumpToDC() {
      this.map.jumpTo({ center: [-77.0628101291, 38.8846868585], zoom: 13 });
    },
    clusterClick(event) {
      const e = event.mapboxEvent;
      const zoom = this.map.getZoom();
      console.log(e.lngLat, zoom);
      this.map.easeTo({ center: e.lngLat, zoom: zoom + 1 });
    },
    circleClick(event) {
      console.log(event);
    },
    onMouseEnter() {
      this.map.getCanvas().style.cursor = "pointer";
    },
    onMouseLeave() {
      this.map.getCanvas().style.cursor = "";
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
.mgl-map-wrapper {
  height: 1000px;
}
.mapboxgl-popup-content {
  background-color: yellow;
}
</style>
