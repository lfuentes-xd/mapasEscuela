<template>
  <div class="container">
    <div class="row">
      <div class="col-sm-4">
        <button class="btn btn-success boton" name="btn_buscar" id="btn_buscar" @click="buscar()">
          buscar
        </button>
        <input type="text" id="txt_bus" v-model="idBusqueda">
      </div>
    </div>
    <br>
    <div class="row">
      <div class="col-6">
        <div id="map"></div>
      </div>
      <div class="col-6 align-left">
        <ul>
          <!-- <li v-for="(escuela, id) in escuelas" :key="id"> -->
          <li v-if="escuelaEncontrada">
            <span>Datos sobre: {{ escuelaEncontrada.nombre }}</span> <br>
            <span>Ubicada en: {{ escuelaEncontrada.Direccion }}</span> <br>
            <span>Telefono: {{ escuelaEncontrada.telefono }}</span><br>
            <span>Horario: {{ escuelaEncontrada.horario }}</span><br>
            <br><br>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
import 'ol/ol.css';
import { Map, View } from 'ol';
import TileLayer from 'ol/layer/Tile';
import OSM from 'ol/source/OSM';
import { fromLonLat } from 'ol/proj';
import Feature from 'ol/Feature';
import Point from 'ol/geom/Point';
import { Icon, Style } from 'ol/style';
import VectorLayer from 'ol/layer/Vector';
import VectorSource from 'ol/source/Vector';
import escuelasArchivo from './escuelas.json';

const lon = -102.28259;
const lat = 21.88234;

export default {
  name: 'escuelas',
  data() {
    return {
      escuelas: escuelasArchivo,
      idBusqueda: '',
      escuelaEncontrada: null,
      escuelalon: null,
      escuelalat: null,
      map: null,
      view: null,
      marcador: null,
    }
  },
  methods: {
    buscar() {
      const id = parseInt(this.idBusqueda);
      this.escuelaEncontrada = this.escuelas.find(escuela => escuela.id === id);
      if(this.escuelaEncontrada){
      this.escuelalon = this.escuelaEncontrada.longitud;
      this.escuelalat = this.escuelaEncontrada.latitud;
      this.actualizarMapa();
      }else{
        alert("Escuela no encontrada");
      }
    },
    actualizarMapa() {
      this.view.setCenter(fromLonLat([this.escuelalon, this.escuelalat]));
      this.marcador.setGeometry(new Point(fromLonLat([this.escuelalon, this.escuelalat])));
    },
    inicializarMapa() {
      this.map = new Map({
        target: 'map',
        layers: [
          new TileLayer({
            source: new OSM()
          })
        ]
      });
      this.view = new View({
        center: fromLonLat([lon, lat]),
        zoom: 16
      });
      this.map.setView(this.view);
      this.marcador = new Feature({
        geometry: new Point(fromLonLat([lon, lat])),
      });
      this.marcador.setStyle(new Style({
        image: new Icon({
          src: "https://img.icons8.com/ios/50/null/marker--v1.png",
        })
      }));
      const capa = new VectorLayer({
        source: new VectorSource({
          features: [this.marcador],
        }),
      });
      this.map.addLayer(capa);
    }
  },
  mounted() {
    this.inicializarMapa();
  }
};

// mounted() {
//   const map = new Map({
//     target: 'map',
//     layers: [
//       new TileLayer({
//         source: new OSM()
//       })
//     ],
//     view: new View({
//       center: fromLonLat([this.escuelalon, this.escuelalat]),
//       zoom: 16
//     })
//   });

//   const marcador = new Feature({
//     geometry: new Point(fromLonLat([this.escuelalon, this.escuelalat])),
//   });

//   marcador.setStyle(new Style({
//     image: new Icon({
//       src: "https://img.icons8.com/ios/50/null/marker--v1.png",
//     })
//   }));

//   const capa = new VectorLayer({
//     source: new VectorSource({
//       features: [marcador],
//     }),
//   });

//   map.addLayer(capa);
// }

</script>

<style>
#map {
  height: 400px;
  /* width: 100%; */
}

.align-left {
  text-align: left;
}

.boton {
  float: left;
}
</style>
