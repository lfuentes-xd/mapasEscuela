<template>
  <div class="container">
    <div class="row">
      <div class="col-4 d-flex">
        <button class="btn btn-primary boton" name="btn_buscar" id="btn_buscar" @click="buscar()">
          buscar
        </button>
        <input class="flex-grow-1 esp" type="text" id="txt_bus" v-model="idBusqueda"
          placeholder="Ingrese el nombre de la escuela" required>
      </div>
    </div>
    <div class="col">
      <div class="row">
        <h1 class="fw-bolder">Filtros para la busqueda</h1>
        <select class="form-select tama" v-model="selectTipo" id="tipo">
          <option value="seleccione el tipo de educacion">seleccione tipo</option>
          <option value="PÚBLICO">Publica</option>
          <option value="PRIVADO">Privada</option>
        </select>

        <select class="form-select tama" v-model="selectNivel" id="nivel">
          <option value="seleccione nivel de educacion">seleccione nivel de educacion</option>
          <option value="INICIAL">Instancias infantiles</option>
          <option value="PREESCOLAR">Preescolar</option>
          <option value="PRIMARIA">Primaria</option>
          <option value="SECUNDARIA">Secundaria</option>
          <option value="MEDIA SUPERIOR">Preparatoria</option>
          <option value="SUPERIOR">Superior</option>
          <option value="CAPACITACION PARA EL TRABAJO">Capacitacion para el trabajo</option>
        </select>

        <input type="submit" class="btn btn-primary tama" value="Filtrar" @click="verificar()">
      </div>
    </div>

    <br>
    <div class="row">
      <div class="col-6">
        <div id="map"></div> <!-- aca se carga el mapa-->
      </div>

      <div class="col-6 align-left"> <!-- aca se cargan los datos de las escuelas-->
        <h1 class="fw-bolder">Datos de la escuela buscada</h1>
        <ul>
          <li v-if="escuelaEncontrada">
            <span>CLAVE_CCT: {{ escuelaEncontrada.CLAVE_CCT }}</span><br>
            <span>TURNO: {{ escuelaEncontrada.TURNO }}</span><br>
            <span>NIVEL: {{ escuelaEncontrada.NIVEL }}</span><br>
            <span>SOSTENIMIENTO: {{ escuelaEncontrada.SOSTENIMIENTO }}</span><br>
            <span>NOMBRE: {{ escuelaEncontrada.NOMBRE }}</span><br>
            <span>CALLE: {{ escuelaEncontrada.CALLE }}</span><br>
            <span>NUMERO: {{ escuelaEncontrada.NUMERO }}</span><br>
            <span>COLONIA: {{ escuelaEncontrada.COLONIA }}</span><br>
            <span>LOCALIDAD: {{ escuelaEncontrada.LOCALIDAD }}</span><br>
            <span>MUNICIPIO: {{ escuelaEncontrada.MUNICIPIO }}</span>
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
  data() { //son los datos que se retornarán.
    return {
      escuelas: escuelasArchivo, //archivo
      idBusqueda: '', //este se obtiene desde arriba, en un input txt_bus
      escuelaEncontrada: null, //en caso de encontrar la escuela
      escuelalon: null, //longitud
      escuelalat: null, //latitud
      map: null, //para inicializar el mapa
      view: null, //para inicializar la vista
      marcador: null, //para poner el marcador
      selectTipo: 'seleccione el tipo de educacion', //para obtener el tipo de escuela
      selectNivel: 'seleccione nivel de educacion',// para obtener el nivel de la misma 
    }
  },
  methods: { //metodos 
    buscar() {
      var nombre = document.getElementById("txt_bus").value.toUpperCase();
      if(nombre.trim()===""){
          alert("por favor coloca el nombre de alguna escuela"); 
      }else{
      // var nombre = document.getElementById("txt_bus").value.toUpperCase();
      this.escuelaEncontrada = this.escuelas.find(escuela => escuela.NOMBRE.toUpperCase().includes(nombre));
      if (this.escuelaEncontrada) {
        this.escuelalon = this.escuelaEncontrada.LONGITUD;
        this.escuelalat = this.escuelaEncontrada.LATITUD;
        this.actualizarMapa();
      } else {
        alert("Escuela no encontrada");
      }
    }
    },
    verificar() {
      var dato1 = document.getElementById("tipo").value;
      var dato2 = document.getElementById("nivel").value;


      if (dato1.trim() === "seleccione tipo" || dato2.trim() === "seleccione el tipo de educacion") {
        //por si el usuario no mete ningun filtro
        alert("seleccione algun filtro primero");
        return false;
      } else if (dato1.trim() === "seleccione el tipo de educacion") {
        //este es por si quieren todas las escuelas no importa si es publica oh no 
        this.filtrar2();
        return false;
      } else if (dato1.trim() != "seleccione el tipo de educacion" && dato2.trim() === "seleccione nivel de educacion") {

        alert("seleccione algun nivel de educacion");
      } else {
        //sino se cumplen las anteriores filtro por nivel y tipo 
        this.filtrar();
      }
    },
    filtrar() {
      var tipo = this.selectTipo;
      var nivel = this.selectNivel;
      this.escuelasFiltradas = [];
      for (var i = 0; i < this.escuelas.length; i++) {
        var escuela = this.escuelas[i];
        if (escuela.SOSTENIMIENTO === tipo && escuela.NIVEL === nivel) {
          this.escuelasFiltradas.push(escuela);
        }
      }

      if (this.escuelasFiltradas.length > 0) {
        const coordenadas = this.escuelasFiltradas.map(escuela => {
          const longitud = parseFloat(escuela.LONGITUD);
          const latitud = parseFloat(escuela.LATITUD);
          return fromLonLat([longitud, latitud]);
        });
        this.actualizarMapa2(coordenadas);
      } else {
        alert("No se encontraron escuelas");
      }
    },
    filtrar2() {
      var tipo = "ninguno seleccionado";
      var nivel = this.selectNivel;
      this.escuelasFiltradas = [];
      alert("filtrando por: \n"
        + "tipo de educacion : " + tipo + "\n"
        + "nivel de educacion: " + nivel);

      for (var i = 0; i < this.escuelas.length; i++) {
        var escuela = this.escuelas[i];
        if (escuela.NIVEL === nivel) {
          this.escuelasFiltradas.push(escuela);
        }
      }

      if (this.escuelasFiltradas.length > 0) {
        const coordenadas = this.escuelasFiltradas.map(escuela => {
          const longitud = parseFloat(escuela.LONGITUD);
          const latitud = parseFloat(escuela.LATITUD);
          return fromLonLat([longitud, latitud]);
        });
        this.actualizarMapa2(coordenadas);
      } else {
        alert("No se encontraron escuelas");
      }

    },
    actualizarMapa2(coordenadas) {
      // Limpia cualquier marcador existente en el mapa
      if (this.capa && this.capa.getSource() && this.capa.getSource().getFeatures().length > 0) {
        this.capa.getSource().clear(); // Limpia los marcadores existentes
      }
      // creamos un vector para los marcadores. 
      const marcadores = [];

      for (let i = 0; i < coordenadas.length; i++) { //<-- mostramos todas las coincidencias 
        const coordenada = coordenadas[i]; //tomamos las coordenadas
        const marcador = new Feature({
          geometry: new Point(coordenada),
        });
        marcador.setStyle(new Style({
          image: new Icon({
            src: "https://img.icons8.com/ios/50/null/marker--v1.png",
          }),
        }));
        marcadores.push(marcador);
      }

      // Crea una nueva capa vectorial con los marcadores
      this.capa = new VectorLayer({
        source: new VectorSource({
          features: marcadores,
        }),
      });

      // Agrega la capa al mapa
      this.map.addLayer(this.capa);
    },
    actualizarMapa() {  //se llama este metodo para actualizar las coordenadas de la escuela
      if (this.capa && this.capa.getSource() && this.capa.getSource().getFeatures().length > 0) {
        this.capa.getSource().clear(); // Limpia los marcadores existentes
      }
      this.view.setCenter(fromLonLat([this.escuelalon, this.escuelalat]));
      this.marcador.setGeometry(new Point(fromLonLat([this.escuelalon, this.escuelalat])));
    },
    inicializarMapa() { //para que se pueda ver el mapa 
      this.map = new Map({
        target: 'map', // en donde se va a cargar
        layers: [
          new TileLayer({
            source: new OSM()
          })
        ]
      });
      this.view = new View({ //se cargan las coordenadas
        center: fromLonLat([lon, lat]),
        zoom: 15
      });
      this.map.setView(this.view); //se pone el marcador
      this.marcador = new Feature({
        geometry: new Point(fromLonLat([lon, lat])),
      });
      this.marcador.setStyle(new Style({ //se pone el icono
        image: new Icon({
          src: "https://img.icons8.com/ios/50/null/marker--v1.png",
        })
      }));
      const capa = new VectorLayer({ //capa del mapa 
        source: new VectorSource({
          features: [this.marcador],
        }),
      });
      this.map.addLayer(capa); //se agrega al mapa
    }
  },
  mounted() {
    this.inicializarMapa(); // se carga 
  }
};
</script>

<style>
#map {
  height: 400px;
}

.align-left {
  text-align: left;
}

.boton {
  float: left;
}

.tama {
  width: 30%;
  margin-bottom: 10px;
}

.esp {
  margin-left: 10px;
}
</style>
