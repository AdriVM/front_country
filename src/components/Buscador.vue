<template>
  <div class="container">
    <!--
        Al hacer submit, llamamos a nuestro metodo buscarPaises para iniciar la busqueda
    -->
    <form @submit.prevent="buscarPaises()">
      <div class="form-group mx-sm-3">
        <!--
                Input que nos servirá para hacer la búsqueda
            -->
        <input
          type="text"
          class="input"
          name="q"
          placeholder="Buscar Paises"
          v-model="q"
        />
        <!--
                Mensaje de Error en la validación
            -->
        <div v-if="enviado && !$v.q.required" class="error">
          Debe ingresar algún dato en el buscador
        </div>
      </div>
      <!--
            Botón Submit
        -->
      <button type="submit" class="btn btn-primary">Buscar</button>
    </form>
    <!--
        La Lista se mostrará cuando se haya rellenado el objeto 'countries' y nuestro array cib kas fronteras que tenga algo.
    -->
    <div v-if="countries.length && borders.length" class="lista">
      <!--
            Llamamos a nuestra función para ordenar el array que contiene nuestras fronteras según el indice que contenga.
        -->
      {{ ordenarFronteras() }}
      <h3>Listado de países según la búsqueda: "{{ q }}"</h3>
      <table v-if="borders" class="tabla">
        <thead>
          <tr>
            <th>Bandera</th>
            <th>Nombre</th>
            <th>Capital</th>
            <th>Región</th>
            <th>Lengua Nativa</th>
            <th>Denominación</th>
            <th>Dominio de Internet</th>
            <th>Área</th>
            <th>Población</th>
            <th>Moneda</th>
            <th>Fronteras</th>
          </tr>
        </thead>
        <!--
                Con vFor recorremos nuestro objeto countries y mostramos los datos en la tabla
            -->
        <tbody>
          <tr v-for="(item, index) in countries" :key="index">
            <td>
              <img :src="item.flag" :alt="item.name" width="50px" />
            </td>

            <td>{{ item.name != "" ? item.name : "-" }}</td>
            <td>{{ item.capital != "" ? item.capital : "-" }}</td>

            <td>{{ item.region != "" ? item.region : "-" }}</td>
            <td>
              {{
                item.languages[0].nativeName != ""
                  ? item.languages[0].nativeName
                  : "-"
              }}
            </td>

            <td>{{ item.demonym != "" ? item.demonym : "-" }}</td>
            <td>{{ item.topLevelDomain != "" ? item.topLevelDomain : "-" }}</td>

            <td v-if="item.area != '' && item.area != null">{{ item.area }}</td>
            <td v-if="item.area == null || item.area == ''"> - </td>
            <td>{{ item.population != "" ? item.population : "-" }}</td>

            <td>
              {{
                item.currencies[0].name != ""
                  ? item.currencies[0].name + " - " + item.currencies[0].symbol
                  : "-"
              }}
            </td>
            <td v-if="item.borders != '' && borders[index].indice == index">
              <!--
                            Recorremos nuestro array de fronteras según el país (indicado mediante el index)
                            y sacamos el nombre de la frontera (pais)
                        -->
              <div
                v-for="(pais, j) in borders[index].fronteras"
                :key="j"
                style="display: inline-block"
              >
                <!--
                                Creamos un botón que mostrará el nombre del país que hace frontera.
                                En el onclick del botón, llamamos a nuestro método buscarMedianteFrontera
                                pasándole el nombre del país para iniciar una nueva búsqueda con dicho nombre.
                            -->
                <button
                  v-on:click="buscarMedianteFrontera(pais)"
                  class="btn-frontera"
                >
                  {{ pais }}
                </button>
              </div>
            </td>
            <td v-if="item.borders == ''">-</td>
          </tr>
        </tbody>
      </table>
    </div>
    <!--
        Si nuestra búsqueda, no devuelve ningún país, mostramos el mensaje.
    -->
    <p v-if="countries.length == 0">
      No hay resultados que coincidan con la búsqueda.
    </p>
  </div>
</template>

<script>
// Importamos Axios para hacer las llamadas HTTP a nuestra API.
import axios from "axios";
// Importamos Vuelidate para validar nuestro buscador.
import { required } from "vuelidate/lib/validators";

export default {
    //Validaciones
    validations: {
        // Indicamos que nuestra búsqueda 'q' es obligatoria.
        q: {
            required,
        },
    },
    data() {
        return {
            // Iniciamos las propiedades que vamos a usar.

            // La propiedad 'enviado' nos va a indicar cuando hemos ejecutado el formulario.
            enviado: false,
            // El parámetro 'q' guardará nuestra búsqueda, que será lo que reciba nuestra API.
            q: "",
            // Array que contendrá los objetos que nos guardarán el índice (que indica el país) y un array con sus fronteras.
            borders: [{
                indice: 0,
                fronteras: []
            }],

            // El parámetro 'countries' nos guardará la respuesta de nuestra API.
            countries: {},
        };
    },
    methods: {

        /**
            Método que recibe los códigos de las fronteras y el indice (que indica a qué país pertenece).
        **/
        mostrarFronteras(alpha3Codes, index){
            // Si el index recibido es 0, eliminamos el elemento con el que lo inicializamos
            /**
                Esto se hizo con la esperanza de quitar el error que
                indicaba que nuestras propiedades eran nulas para los
                objetos que formaban el array de borders.
            **/
            if(index == 0){
                this.borders.pop();
            }

            // Si recibimos los códigos de las fronteras, procedemos a la búsqueda de sus nombres.
            if (alpha3Codes != "") {
                // Array temporal
                var temp = [];
                axios
                    .get("http://127.0.0.1:8000/api/borders/" + alpha3Codes)
                    .then((response) => {
                        /**
                            Recorremos el 'response' para guardar tan solo
                            el NOMBRE de las fronterasen el array temporal.
                        **/
                        for (let i = 0; i < response.data.length; i++) {
                            temp.push(response.data[i].name);
                        }
                        /**
                         *  Una vez nuestro array temporal está relleno, guardamos
                         * las fronteras y el indice (que indica de qué país son dichas fronteras)
                         * en nuestro array de borders
                         **/
                        this.borders.push({ indice: index, fronteras: temp });
                    });
            /**
             * Si no recibimos códigos porque el país no tiene frontera,
             * guardamos tan sólo el indice para indicar el país, con el
             * array de fronteras, vacío.
            **/
            }else{
                this.borders.push({ indice: index, fronteras: [] });
            }
        },

        /**
         * Método que se encarga de ordenar nuestro array de borders,
         * que almacena los nombres de los países que hacen frontera.
        **/
        ordenarFronteras(){
            this.borders.sort(function (a, b){
                if (a.indice > b.indice) {
                    return 1;
                }
                if (a.indice < b.indice) {
                    return -1;
                }
                return 0;
            });
        },


        /**
            * Método encargado de buscar y guardar los países que le pasamos mediante nuestra variable 'q'
            **/
        buscarPaises(){

            // Marcamos la variable 'enviado' a true, porque hemos hecho click en el botón buscar.
            this.enviado = true;

            // Inicializamos 'borders' y 'countries' para limpiar la tabla de cara a una nueva búsqueda.
            this.borders = [{ indice: 0, fronteras: [] }];
            this.countries = {};

            // Comprobamos la si ha pasado la validación para que NO continue si ha fallado.
            this.$v.$touch();
            if (this.$v.$invalid) {
                return false;
            } else {
                // No ha fallado la validación, continuamos la ejecución.


                        // Hacemos la llamada a nuestra API, pasándole como parámetro nuestra variable 'q'.
                        axios
                            .get("http://127.0.0.1:8000/api/countries/" + this.q)
                            .then((response) => {
                                 console.log(response.data);
                                // Array donde guardaremos el response filtrado por el name.
                                var respuestaAPI = [];

                                // La búsqueda será tanto en mayúsculas como minúsculas para filtrar.
                                const str = this.q;
                                const strMayus = str.charAt(0).toUpperCase() + str.slice(1);

                                // Recorremos el response que nos devuelve nuestra API
                                for (let i = 0; i < response.data.length; i++) {
                                    // Si lo que devuelve el response, contiene en el name (y sólo en el name) nuestra búsqueda.
                                    if (response.data[i].name.includes(strMayus || str)) {
                                        // Lo agregamos a nuestra Array
                                        respuestaAPI.push(response.data[i]);
                                    }
                                }

                                // Guardamos nuestro array filtrado al objeto de countries.
                                this.countries = respuestaAPI;


                                // Recorremos el array para buscar los códigos de las fronteras
                                // y pasárselo a nuestra función 'mostrarFronteras()', junto con el indice.
                                for (let i = 0; i < respuestaAPI.length; i++) {
                                this.mostrarFronteras(respuestaAPI[i].borders, i);
                                }
                            });
            }
        },



        /**
            * Método del onclick del botón de las fronteras, recibe el nombre de la frontera como búsqueda.
            **/
        buscarMedianteFrontera(busqueda){
            /**
                * Guardamos el nombre de la frontera en la variable 'q',
                * esto hace que se pinte en nuestro input.
                **/
            this.q = busqueda;

            // Ejecutamos el método de 'buscarPaises()'
            this.buscarPaises();
        },

    },
    
};
</script>


<!--
    ESTILOS
-->

<style>
input {
  margin: 0;
  padding: 0;
  width: auto;
}

.container {
  width: 90%;
  padding-right: 15px;
  padding-left: 15px;
  margin-right: auto;
  margin-left: auto;
  text-align: center;
}

.form-group {
  margin-bottom: 1rem;
}

@media (min-width: 576px) {
  .mx-sm-3 {
    margin-left: 1rem !important;
  }
}

.input {
  background: none;
  border: none;
  height: 75px;
  padding: 0px 0px 0px 25px;
  font-size: 30px;
  border-radius: 40px;
  box-shadow: inset -3px -3px 8px #b1b1b1, 3px 3px 8px #ffffff;
  width: 400px;
}

.input:focus-visible {
  outline-style: none;
  border: 1px solid grey;
  border-radius: 40px;
  box-shadow: inset -3px -3px 8px #b1b1b1, 3px 3px 8px #ffffff;
}

.btn {
  outline: none;
  border: none;
  cursor: pointer;
  width: 25%;
  height: 60px;
  margin-top: 10px;
  border-radius: 30px;
  font-size: 20px;
  font-family: "Lato", sans-serif;
  color: #fff;
  text-align: center;
  background-color: #41b883;
  box-shadow: 3px 3px 8px #b1b1b1, -3px -3px 8px #ffffff;
  transition: 0.5s;
}

.btn:hover,
.btn-frontera:hover {
  background-color: #43cb89;
}

.btn:not(:disabled):not(.disabled) {
  cursor: pointer;
}

.btn-frontera {
  outline: none;
  border: none;
  cursor: pointer;
  padding: 8px;
  margin: 5px;
  border-radius: 30px;
  font-weight: 700;
  font-family: "Lato", sans-serif;
  color: #fff;
  text-align: center;
  background-color: #35495e;
  box-shadow: 3px 3px 8px #b1b1b1, -3px -3px 8px #ffffff;
  transition: 0.5s;
}

.error {
  color: red;
  margin-top: 1rem;
}

.tabla {
  margin: 0 auto;
  margin-bottom: 20px;
  box-shadow: 6px 6px 12px #b8b9be, -6px -6px 12px #fff !important;
  padding: 25px;
  border-radius: 20px;
}

.tabla tr td {
  border-top: 0.0625rem solid #d1d9e6;
}

@media only screen and (max-width: 760px), (min-device-width: 768px) and (max-device-width: 1024px) {
  table,
  thead,
  tbody,
  th,
  td,
  tr {
    display: block;
  }

  thead tr {
    position: absolute;
    top: -9999px;
    left: -9999px;
  }

  tr:nth-child(even) {
    background: #ececed;
  }

  tr:first-child td:first-child {
    border-top: none !important;
  }

  td {
    border: none;
    position: relative;
    padding-left: 50%;
  }

  td:before {
    position: absolute;
    top: 0;
    left: 6px;
    width: 45%;
    padding-right: 10px;
    white-space: nowrap;
  }

  td:nth-of-type(1):before {
    content: "Bandera";
    font-weight: bold;
  }
  td:nth-of-type(2):before {
    content: "Nombre";
    font-weight: bold;
  }
  td:nth-of-type(3):before {
    content: "Capital";
    font-weight: bold;
  }
  td:nth-of-type(4):before {
    content: "Región";
    font-weight: bold;
  }
  td:nth-of-type(5):before {
    content: "Lengua Nativa";
    font-weight: bold;
  }
  td:nth-of-type(6):before {
    content: "Denominación";
    font-weight: bold;
  }
  td:nth-of-type(7):before {
    content: "Dominio Internet";
    font-weight: bold;
  }
  td:nth-of-type(8):before {
    content: "Área";
    font-weight: bold;
  }
  td:nth-of-type(9):before {
    content: "Población";
    font-weight: bold;
  }
  td:nth-of-type(10):before {
    content: "Moneda";
    font-weight: bold;
  }
  td:nth-of-type(11):before {
    content: "Fronteras";
    font-weight: bold;
  }
}
</style>
