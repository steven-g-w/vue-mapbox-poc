<template>
  <div class="hello">
    <h1>POC Map</h1>
    <button v-on:click="stelliteOn">stelliteOn</button>
    <button v-on:click="stelliteOff">stelliteOff</button>
    <button v-on:click="FlyToBrisbane">FlyToBrisbane</button>
    <button v-on:click="EaseToBrisbane">EaseToBrisbane</button>
    <button v-on:click="JumpToBrisbane">JumpToBrisbane</button>
    <button v-on:click="JumpToDC">JumpToDC</button>
    <button @click="updateMarkerData">Update Marker Data</button>
    <!-- :center="[-77.0628101291, 38.8846868585]" -->
    <!-- :zoom="13" -->

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

      <!-- <MglGeojsonLayer
        :sourceId="'geojson-source'"
        :source="geoJsonSource"
        layerId="circle"
        :layer="geojsonPrettierCircleLayer"
        @mouseenter="showCircleDetail"
        @click="circleClick"
      /> -->

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
        v-for="(m, index) in markers"
        :key="index"
        :coordinates="m.geometry.coordinates"
        @click="() => markerClicked(m)"
      >
        <div slot="marker">
          <div class="marker">
            <h1 :style="`background-color: ${m.properties.color || 'white'}`">
              <!-- {{ m.properties.description }} -->
            </h1>
          </div>
        </div>
      </MglMarker>
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
      markers: [],
      accessToken:
        "pk.eyJ1IjoidG9ueS13b25nLWF1cml6b24iLCJhIjoiY2tnaWhxb2VoMDZneDMxcGR5ZmNmaHVtayJ9.ZkvcS3ihI98gUa-AQ4UeeQ", // your access token. Needed if you using Mapbox maps
      mapStyle: "mapbox://styles/tony-wong-aurizon/ckgij4stc06ln1amvev3cvmhb", // your map style
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
          "icon-image": "white",
          "icon-text-fit": "both",
          "icon-text-fit-padding": [1, 5, 1, 5],
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
          // "circle-radius": 0,
          "text-halo-width": 1,
          "text-halo-blur": 0,
          "text-halo-color": "#846cff",
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
  watch: {
    map: {
      deep: true,
      handle() {
        console.log("map change");
        this.updateMarkers("map change");
      },
    },
  },
  methods: {
    onMapLoaded(event) {
      // in component
      console.log("onMapLoaded");
      this.map = event.map;
      this.map.center = [153.04, -27.47];

      this.map.loadImage("/white.png", (error, image) => {
        if (error) throw error;
        this.map.addImage("white", image);
      });

      this.map.on("moveend", () => this.updateMarkers("moveend"));
      this.updateMarkers("load");
    },
    markerClicked(m) {
      m.properties.color = "green";
    },
    updateMarkers(a) {
      console.log(a);

      this.markers = [];

      // wait for the map to load.

      const features = this.map.querySourceFeatures("geojson-source");

      console.log(features);

      features.forEach((f) => {
        if (!f.properties.cluster) {
          this.markers.push(f);
        }
      });
    },
    updateMarkerData() {
      this.updateMarkers();
      this.markers = this.markers.filter((m, index) => {
        m.properties.color = "red";
        return index % 2;
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
      this.map.easeTo({ center: [-77.0628101291, 38.8846868585], zoom: 13 });
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
<style>
.marker {
  background: white;
  border-radius: 50%;
  margin: 10px;
  height: 20px;
  width: 20px;

  box-shadow: 0 0 0 0 rgba(255, 255, 255, 1);
  transform: scale(1);
  animation: pulse 2s infinite;
}
@keyframes pulse {
  0% {
    transform: scale(0.95);
    box-shadow: 0 0 0 0 rgba(255, 255, 255, 0.7);
  }

  70% {
    transform: scale(1);
    box-shadow: 0 0 0 10px rgba(255, 255, 255, 0);
  }

  100% {
    transform: scale(0.95);
    box-shadow: 0 0 0 0 rgba(255, 255, 255, 0);
  }
}
</style>
