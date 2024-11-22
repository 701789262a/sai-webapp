<template>
  <div id="map" :style="{ width: '100%', height: '100%' }"></div>
</template>

<script>
import L from 'leaflet';
import { store } from '/src/components/store.js';
import Popup from './Popup.vue';
import { createApp } from 'vue';
import puntook from "/src/assets/pngs/puntook.png";

export default {
  name: 'Map',
  data() {
    return {
      map: null,
      geoJsonLayer: null,
      layergroup: null, // Manage markers in a group
      themes: {
        "AMBIENTE E AGRICOLTURA": "g1",
        "ASSETTO IDROGEOLOGICO": "g2",
        "PAESAGGIO - Parte III del D.Lgs 42/2004 - Art. 136 e 157": "g3",
        "PAESAGGIO - Parte III del D.Lgs 42/2004 - Art. 142 - Aree tutelate per legge": "g4",
        "PAESAGGIO - Parte III del D.Lgs 42/2004 - Art. 143 comma 1 lettera d": "g5",
        "ULTERIORI CONTESTI BENI IDENTITARI - Parte III del D.Lgs 42/2004 - Art. 143 comma 1 lettera e": "g6",
        "SITI UNESCO": "g7",
      },
      impianto: {
        'Fotovoltaico e Termodinamico': 'FOTOTERMO',
        'Eolico': 'EOLICO',
        'Biomasse': 'BIOMASSE',
        'Idroelettrico': "IDROELETTRICO",
        'Geotermico': 'GEOTERMOELETTRICO',
      },
      dimensione: {
        'Piccola taglia': "PICCOLA TAGLIA",
        'Media taglia': 'MEDIA TAGLIA',
        'Grande taglia': 'GRANDE TAGLIA',
        'Micro taglia': 'MICRO TAGLIA',
        'Mini taglia': 'MINI TAGLIA',
        'Micro eolico': 'MICRO EOLICO',
        'Mini eolico': 'MINI EOLICO',
        'Grande eolico (Eolico)': "EOLICO",
        'Bassa entalpia': 'BASSA ENTALPIA',
        'Media entalpia': 'MEDIA ENTALPIA',
        'Alta entalpia': 'ALTA ENTALPIA',
      },
      markerIcon: new L.Icon({
        iconUrl: puntook,
        iconSize: [24, 36],
        iconAnchor: [12, 36],
        popupAnchor: [0, -36],
      }),
    };
  },
  mounted() {
    this.initializeMap();

    this.$watch(
      () => store.layer,
      () => {
        this.loadGeoJsonLayer(store.layer);
      },
      { immediate: true }
    );
  },
  methods: {
    initializeMap() {
      // Initialize map
      this.map = L.map('map', {
        zoomControl: false,
        scrollWheelZoom: 'center',
      }).setView([40.1209, 9.1217], 8);

      // Add tile layer
      L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png').addTo(this.map);

      // Initialize layer group for markers
      this.layergroup = L.layerGroup().addTo(this.map);

      // Handle map click events
      this.map.on('click', this.onMapClick);
    },
    async onMapClick(e) {
      try {
        this.layergroup.clearLayers(); // Clear previous markers
      } catch (error) {
        console.error('Error clearing layers:', error);
      }

      // Extract details
      const tipo = store.layer.split('_')[0];
      const dim = store.layer.split('_')[1];
      const _tipo = this.impianto[tipo];
      const _dimensione = this.dimensione[dim];

      // Fetch API data
      const response = await fetch(
        `https://sai.zeromist.net/api/getpoint?x=${e.latlng.lng}&y=${e.latlng.lat}&tipo=${_tipo}&dimensione=${_dimensione}`
      );
      const data = await response.json();

      // Check for valid data
      if (!store.layer) return;

      // Create popup content
      const popupContainer = document.createElement('div');
      const popupApp = createApp(Popup, { j: data });
      popupApp.mount(popupContainer);

      // Add marker to map
      const marker = L.marker([e.latlng.lat, e.latlng.lng], {
        icon: this.markerIcon,
      }).bindPopup(popupContainer, { closeButton: false });

      // Add marker to layer group
      this.layergroup.addLayer(marker);
    },
    loadGeoJsonLayer(layerPath) {
      if (!layerPath) return;

      // Remove existing GeoJSON layer if present
      if (this.geoJsonLayer) {
        this.map.removeLayer(this.geoJsonLayer);
      }

      const tipo = layerPath.split('_')[0];
      const dim = layerPath.split('_')[1];
      const _tipo = this.impianto[tipo];
      const _dimensione = this.dimensione[dim];

      const wmsOptions = {
        service: 'WMS',
        request: 'GetMap',
        layers: `SAI:IMPIANTO ${_tipo} ${_dimensione}`,
        srs: 'EPSG:3003',
        format: 'image/png',
        transparent: true,
        styles: 'sai_colors',
        tiled: true,
      };

      this.geoJsonLayer = L.tileLayer.wms(
        'https://sai.zeromist.net/geoserver/SAI/wms',
        wmsOptions
      ).addTo(this.map);
    },
  },
};
</script>

<style scoped>
#map {
  position: absolute;
  z-index: 998;
}

::v-deep(.leaflet-popup-content-wrapper) {
  background-color: #000;
}

::v-deep(.leaflet-popup-tip) {
  width: 0px;
  height: 0px;
}

::v-deep(.svg-fill) {
  filter: invert(1);
  color: gray;
}
</style>
