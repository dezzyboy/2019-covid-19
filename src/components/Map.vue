<template>
  <l-map
    ref="map"
    style="z-index: 0; height: 100%; width: 100%"
    :zoom="zoom"
    :center="center"
    @update:zoom="zoomUpdated"
    @update:center="centerUpdated"
    @update:bounds="boundsUpdated"
  >
    <l-tile-layer :url="tileLayerUrl"></l-tile-layer>
    <l-circle-marker
      @click="$emit('MARKER_CLICKED', l)"
      v-for="(l, idx) in locations"
      :key="idx"
      :lat-lng="[+l.Lat, +l.Long]"
      color="#e84118"
      fillColor="#e84118"
      :fillOpacity="0.35"
      :stroke="false"
      :radius="l.radius"
    />
  </l-map>
</template>

<script>
import { LMap, LTileLayer, LCircleMarker } from 'vue2-leaflet';
import { mapState } from 'vuex';

export default {
  name: 'Map',
  props: ['data'],
  data: () => ({
    url: 'https://tiles.stadiamaps.com/tiles/alidade_smooth_dark/{z}/{x}/{y}{r}.png',
    zoom: 2,
    center: [35.000074, 104.999927],
    bounds: null
  }),
  computed: {
    ...mapState(['isDarkTheme']),
    locations() {
      const withConfirmedData = this.data.filter(i => i.dates[i.dates.length - 1].confirmed);
      return withConfirmedData.map(item => ({
        ...item,
        radius: this.scale(item.dates[item.dates.length - 1].confirmed)
      }));
    },
    tileLayerUrl() {
      const darkTheme = 'https://{s}.basemaps.cartocdn.com/dark_all/{z}/{x}/{y}{r}.png';
      const lightTheme = 'https://{s}.basemaps.cartocdn.com/rastertiles/voyager/{z}/{x}/{y}{r}.png';
      return this.isDarkTheme ? darkTheme : lightTheme;
    }
  },
  mounted() {
    this.getUserLocation();
  },
  methods: {
    zoomUpdated(zoom) {
      this.zoom = zoom;
    },
    centerUpdated(center) {
      this.center = center;
    },
    boundsUpdated(bounds) {
      this.bounds = bounds;
    },
    flyTo(lat, lon) {
      this.$refs.map.mapObject.flyTo([lat, lon]);
    },
    scale(d) {
      const min = 1;
      const factor = 5;
      const zoomFactor = this.zoom >= 5 ? 1 : this.zoom / 10; // adjust divisor for best optics
      return Math.floor(Math.log(d) * factor * zoomFactor) + min;
    },
    getUserLocation() {
      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(({ coords }) => {
          this.flyTo(coords.latitude, coords.longitude);
        });
      }
    }
  },
  components: {
    LMap,
    LTileLayer,
    LCircleMarker
  }
};
</script>
