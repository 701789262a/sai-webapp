<template>
  <div id="map" :style="{ width: '100%', height: '100%' }"></div>
</template>

<script>
import L from 'leaflet';
import markerIcon from '@/assets/pngs/puntocheck.png';
import { store } from '/src/components/store.js';

const puntocheck = new L.Icon({
  iconUrl: markerIcon,
  shadowUrl: markerIcon,
  iconSize: [20, 18],
  iconAnchor: [10, 18],
  popupAnchor: [0, -10],
  shadowSize: [0, 0]
});

export default {
  name: 'Map',
  data() {
    return {
      map: null,
      geoJsonLayer: null, // Track the GeoJSON layer
      punti: [],
    };
  },
  mounted() {
    this.initializeMap();
    this.impianto = {
      'Fotovoltaico e Termodinamico': 'FOTOTERMO',
      'Eolico': 'EOLICO',
      'Biomasse': 'BIOMASSE',
      'Idroelettrico': "IDROELETTRICO",
      'Geotermico': 'GEOTERMOELETTRICO'
    }
    this.dimensione = {
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
    }
    // Watch for changes to the layer in the store and reload the GeoJSON
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
      this.map = L.map('map', { zoomControl: false }).setView([40.1209, 9.1217], 8);
      L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png').addTo(this.map);

      this.punti.forEach(punto => {
        L.marker([punto.lat, punto.lng], { icon: puntocheck }).addTo(this.map)
          .bindPopup("<h3>Dettagli Vincoli</h3><p>Descrizione lunga...</p>")
          .openPopup();
      });

      this.loadGeoJsonLayer(store.layer); // Load initial layer
    },
    loadGeoJsonLayer(layerPath) {
      if (layerPath == "") {
        return
      }
      // Remove existing layer if present
      if (this.geoJsonLayer) {
        this.map.removeLayer(this.geoJsonLayer);
      }
      var tipo = layerPath.split('_')[0]
      console.log(tipo)
      var dim = layerPath.split('_')[1]
      console.log(dim)
      var _tipo = this.impianto[tipo]
      var _dimensione = this.dimensione[dim]
      console.log(_tipo)
      console.log(_dimensione)

      //console.log(this.impianto[tipo])
      // Load new GeoJSON layer
      var wmsOptions = {
        service: 'WMS',
        request: 'GetMap',
        layers: `SAI:IMPIANTO ${_tipo} ${_dimensione}`,
        srs: 'EPSG:3003',
        format: 'image/png',
        transparent: true,
        styles: 'sai_colors',
        tiled:true
      }
      this.geoJsonLayer = L.tileLayer.wms('https://sai.zeromist.net/geoserver/SAI/wms', wmsOptions).addTo(this.map)
    },

  },
};
</script>

<style scoped>
#map {
  position: absolute;
  z-index: 998;
}
</style>
