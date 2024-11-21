<template>
  <div id="map" :style="{ width: '100%', height: '100%' }"></div>
</template>

<script>
import L from 'leaflet';
import markerIcon from '@/assets/pngs/puntocheck.png';
import { store } from '/src/components/store.js';
import Popup from './Popup.vue';
import { createApp } from 'vue';
import puntook from "/src/assets/pngs/puntook.png";

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
    this.themes = {
      "AMBIENTE E AGRICOLTURA": "g1",
      "ASSETTO IDROGEOLOGICO": "g2",
      "PAESAGGIO - Parte III del D.Lgs 42/2004 - Art. 136 e 157": "g3",
      "PAESAGGIO - Parte III del D.Lgs 42/2004 - Art. 142 - Aree tutelate per legge": "g4",
      "PAESAGGIO - Parte III del D.Lgs 42/2004 - Art. 143 comma 1 lettera d": "g5",
      "ULTERIORI CONTESTI BENI IDENTITARI - Parte III del D.Lgs 42/2004 - Art. 143 comma 1 lettera e": "g6",
      "SITI UNESCO": "g7",
    }
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
    this.markerIcon = new L.Icon({
      iconUrl:
        puntook,
      iconSize: [20, 18],
      iconAnchor: [10, 18],
      popupAnchor: [0, 75],
      shadowSize: [0, 0]
    });
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
      this.layergroup = L.layerGroup()
      this.map = L.map('map', { zoomControl: false }).setView([40.1209, 9.1217], 8);
      L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png').addTo(this.map);

      this.punti.forEach(punto => {
        L.marker([punto.lat, punto.lng], { icon: puntocheck }).addTo(this.map)
          .bindPopup("<h3>Dettagli Vincoli</h3><p>Descrizione lunga...</p>")
          .openPopup();
      });

      this.loadGeoJsonLayer(store.layer); // Load initial layer
      this.layergroup = L.layerGroup()
      this.map.on('click', async (e) => {
        try {
          this.layergroup.clearLayers();
        }
        catch (e) {

        }
        var tipo = store.layer.split('_')[0]
        console.log(tipo)
        var dim = store.layer.split('_')[1]
        console.log(dim)
        var _tipo = this.impianto[tipo]
        var _dimensione = this.dimensione[dim]
        this.res = await fetch(`https://sai.zeromist.net/api/getpoint?x=${e['latlng']['lng']}&y=${e['latlng']['lat']}&tipo=${_tipo}&dimensione=${_dimensione}`)
        this.j = await this.res.json()

        
        if (store.layer == "") {
          return;
        }
        this.popupContainer = document.createElement('div');

        this.popupApp = createApp(Popup, {
          j: this.j,
        });

        this.popupApp.mount(this.popupContainer);

        this.marker = await L.marker([e['latlng']['lat'], e['latlng']['lng']], {
          icon: this.markerIcon,
          name: 'diocan'
        }).bindPopup(this.popupContainer, {closeButton: false})
        // cerca su api
        //this.layergroup.addLayer(this.marker)

        console.log(this.j)
        console.log(e)
        this.layergroup.addLayer(this.marker)
          .addTo(this.map);
        //this.map.addLayer(this.marker)
      });
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
        tiled: true
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

::v-deep(.leaflet-popup-content-wrapper) {
  background-color: #000
}

::v-deep(.leaflet-popup-tip) {
  width: 0px;
  height: 0px;
}

v::deep(.svg-fill) {
  filter: invert(1);
  color: gray;

}
</style>
