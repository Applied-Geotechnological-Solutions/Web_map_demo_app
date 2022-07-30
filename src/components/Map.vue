<template>



 <div class="home-header" style="margin-left: 75vh; margin-top:1vh; margin-bottom:2vh;"> Web Map Demo </div>





  <div class="grid-container">

        <div class="left">

            <div class="toc">



                <h3 class="header-orange" > MAP SETTINGS </h3>

                    <div class="text-bold-white"> Icon size: </div>

                            <input type="range" min="0.3" max="0.8" step="0.1" v-model="icon_size" @change=change_icon_scale(icon_size) style='width:13vh;'>

                            <br>

                    <div class=text-bold-white>
                                Base layers:
                            </div>

                    <div v-for="item in layer_items" :key="item" class="overlay-text-feed">

                        <div v-if="item.base_layer">

                            <input type="radio" :value="item.src" @change=change_baselayer() v-model="Baselayer_src">

                            <label :for="item.name">{{item.name.replace(/_/g, " ")}}</label> <br>

                        </div>

                    </div>




                <h3 class="header-orange" > DATA LAYERS </h3>



                        <div v-for="layer in layer_groups" :key="layer" class="overlay-text-feed">


                            <div v-if="layer.name != 'Base layer'">







                                <input type="checkbox" :id="layer.name" :value="layer.name" v-model="layer.visible" @change="change_layer_group_visibility(layer.name, layer.visible)">

                                <label class="text-bold-white" :for="layer.name">{{layer.name}}</label>

                                <div v-for="item in layer_items" :key="item" class="overlay-text-feed">

                                <div v-if="item.layer_group == layer.name">

                                <li>
                                    <input type="checkbox" :id="item.visible" :value="item.visible" @change="change_layer_visibility(item.name, layer.visible)" v-model="item.visible">

                                    <label :for="item.name">{{item.name.replace(/_/g, " ")}}</label> <br>

                                </li>


                            </div>


                        </div>
                        <br>

                  </div>

                </div>


                </div>

              </div>

      <div class="right">

        <div id="map" class="map__x" ref="mapCom"></div>

        <button @click='toggle = !toggle' ref="legend__toggle_overlay" class="overlay-legend-toggle"> Legend </button>

        <div v-show='toggle' ref="legend_overlay" class="overlay-legend">

            <div class=legend-header>
                         Legend
                      </div>


                <div v-for="item in layer_items" :key="item" class="legend-list">

                  <div v-if="item.visible & item.inlegend">

                    <div class=legend-grid-container>

                      <div class=legend-left>

                        <img :src="item.img" class="legend-img">

                      </div>

                      <div class=legend-right>
                        <div class=legend-text>
                           {{item.name.replace(/_/g, " ")}}
                        </div>

                      </div>

                    </div>

                  </div>

                </div>

        </div>

        <div ref="popupMessage1" class="popup2">

          <div class=popup2-text>
            Icons and design layout will be costumized for clients' needs!
          </div>





          <span class="popup-close2" @click="closePopup2">x</span>

        </div>


          <div ref="popupAttributes" class="popup">

              <span class="popup-close" @click="closePopup1"> x </span>

              <div class="header-orange-attributes" > Details: </div>
              <hr>

                <div class="overlay-text-info" v-for="item in popup_content_info" :key="item">

                  <div v-html="item"> </div>

                </div>
                <hr style="height: 0.1px; background:#fff;">


              <div class="overlay-text-feed" v-html="popup_content_feed"> </div>

          </div>

        </div>

    </div>


</template>

<script>
export default {
        name: 'Secure',
        data() {
            return {};
        }
    }
</script>

<script setup>


import { ref, onMounted} from 'vue'
import { Map, View } from 'ol' // Introduce container binding module and view module
import XYZ from 'ol/source/XYZ' // introduce XYZ Map format
import Overlay from 'ol/Overlay'// Introduce the cover module
import 'ol/ol.css' // ol Provided css style （ Must introduce
//import {Group as LayerGroup} from 'ol/layer';

import TileLayer from 'ol/layer/Tile'
//import TileWMS from 'ol/source/TileWMS';
import VectorLayer from 'ol/layer/Vector'
import VectorSource from 'ol/source/Vector'
import GeoJSON from 'ol/format/GeoJSON'
import {Icon, Style} from 'ol/style';
import OSM from 'ol/source/OSM'
//import Select from 'ol/interaction/Select';
//import {click} from 'ol/events/condition';
import {OverviewMap, defaults as defaultControls, ZoomSlider} from 'ol/control';
import MousePosition from 'ol/control/MousePosition';
import {createStringXY} from 'ol/coordinate';
import ScaleLine from 'ol/control/ScaleLine';

import LineString from 'ol/geom/LineString.js';

import Feature from 'ol/Feature';
import Point from 'ol/geom/Point';
import {getVectorContext} from 'ol/render';
import {Circle as CircleStyle, Fill, Stroke} from 'ol/style';


//import jsPDF from 'jspdf';

//import PdfPrinter from 'ol-pdf-printer';



//import questionIcon from "@/assets/questionIcon.png";



const map = ref(null) // Examples of maps
//const icons_link = ref('https://github.com/Syverpet/prototyping_data_april_2022/tree/main/icons/') // Github link to icons folder

//const format = ref(undefined)

//const resolution = ref(undefined)

//const dims = ref({
//  1: [1189, 841],
//  2: [841, 594],
//  3: [594, 420],
//  4: [420, 297],
//  5: [297, 210],
//  6: [210, 148],
//});

//var pdfPrinter = ref(new PdfPrinter(opt_options))
















//const local_host = ref(window.location.protocol + "//" + window.location.host + "/") // local host variable.





// WHENNEWOVERLAY
const mapCom = ref(null) // Map container
const popupAttributes = ref(null) // Pop up containerlocal_host
const popupMessage1 = ref(null) // Pop up container
const legend__toggle_overlay = ref(null) // Pop up container
const legend_overlay = ref(null) // Pop up container
 // WHENNEWOVERLAY
const overlay = ref(null)
const overlay2 = ref(null)
const overlay3 = ref(null)
const overlay4 = ref(null)
const currentCoordinate = ref('') // Pop up information

const popup_content_info = ref('')

const popup_content_feed = ref('')


const keys_list = ref([])


const layers = ref(null)

const viewResolution = ref(null)



const selected = ref(null)

//const selectStyle = new Style({
//  fill: new Fill({
//    color: '#eeeeee',
//  }),
//  stroke: new Stroke({
//    color: 'rgba(255, 255, 255, 0.7)',
//    width: 2,
//  }),
//});


// Legend list items // WHENADDLAYERS

const layer_groups = ref([
    {name: 'Base layer',
    visible: true},
    {name: 'Health',
    visible: true},
    {name: 'Food Security',
    visible: true},
    {name: 'WASH',
    visible: true},
    {name: 'Assets',
    visible: true},
])

//const group_visibibility = ref({
//    Health: true,
//    Food_Security: true,
//})


const layer_items = ref([
  {
    id: 0,
    name: 'Open_Street_Map',
    img: 'https://github.com//Syverpet/prototyping_data_april_2022/blob/main/icons/Health_Equipment_and_Supplies.png?raw=true',
    visible: true,
    inlegend: false,
    base_layer: true,
    src: new OSM(),
    layer_group: 'base layer',
  },

  {
    id: 1,
    name: 'Google_Satellite',
    img: 'https://github.com//Syverpet/prototyping_data_april_2022/blob/main/icons/Health_Equipment_and_Supplies.png?raw=true',
    visible: true,
    inlegend: false,
    base_layer: true,
    src: new XYZ({
          url: 'http://mt0.google.com/vt/lyrs=y&hl=en&x={x}&y={y}&z={z}'
      }),
    layer_group: 'base layer',
  },

  {
    id: 2,
    name: 'Dark_Basemap',
    img: 'https://github.com//Syverpet/prototyping_data_april_2022/blob/main/icons/Health Facilities.png?raw=true',
    visible: true,
    inlegend: false,
    base_layer: true,
    src: new XYZ({
      url: "http://c.tile.jawg.io/jawg-dark/{z}/{x}/{y}.png?access-token=87PWIbRaZAGNmYDjlYsLkeTVJpQeCfl2Y61mcHopxXqSdxXExoTLEv7dwqBwSWuJ",
      crossOrigin: undefined,
      view: new View({ projection: 'EPSG:4326', // Projection coordinate system
        center: [113.1206, 23.034996], // The center of the map
        zoom: 1, // Map zoom level （ The default level when opening a page,
      }),
    }),
    layer_group: 'base layer',
  },



// FOOD SECURITY

    {
    id: 3,
    name: 'Cash Assistance',
    img: 'https://github.com//Applied-Geotechnological-Solutions/Web_map_demo_data/blob/main/icons/Food%20Security/1.cash.png?raw=true',
    visible: true,
    inlegend: true,
    base_layer: false,
    layer_group: 'Food Security',
  },

  {
    id: 4,
    name: 'Fishing Equipment Distribution',
    img: 'https://github.com//Applied-Geotechnological-Solutions/Web_map_demo_data/blob/main/icons/Food%20Security/2.fish.png?raw=true',
    visible: true,
    inlegend: true,
    base_layer: false,
    layer_group: 'Food Security',
  },

  {
    id: 5,
    name: 'Food Distribution',
    img: 'https://github.com//Applied-Geotechnological-Solutions/Web_map_demo_data/blob/main/icons/Food%20Security/3.food.png?raw=true',
    visible: true,
    inlegend: true,
    base_layer: false,
    layer_group: 'Food Security',
  },

  {
    id: 6,
    name: 'Food Security Assessment',
    img: 'https://github.com//Applied-Geotechnological-Solutions/Web_map_demo_data/blob/main/icons/Food%20Security/4.fass.png?raw=true',
    visible: true,
    inlegend: true,
    base_layer: false,
    layer_group: 'Food Security',
  },

    {
    id: 7,
    name: 'Food Training',
    img: 'https://github.com//Applied-Geotechnological-Solutions/Web_map_demo_data/blob/main/icons/Food%20Security/5.ftraining.png?raw=true',
    visible: true,
    inlegend: true,
    base_layer: false,
    layer_group: 'Food Security',
  },

  {
    id: 8,
    name: 'Farming Equipment Distribution',
    img: 'https://github.com//Applied-Geotechnological-Solutions/Web_map_demo_data/blob/main/icons/Food%20Security/6.fequipment.png?raw=true',
    visible: true,
    inlegend: true,
    base_layer: false,
    layer_group: 'Food Security',
  },

  {
    id: 9,
    name: 'Seed Distribution',
    img: 'https://github.com//Applied-Geotechnological-Solutions/Web_map_demo_data/blob/main/icons/Food%20Security/7.seed.png?raw=true',
    visible: true,
    inlegend: true,
    base_layer: false,
    layer_group: 'Food Security',
  },


  // WASH

{
    id: 10,
    name: 'Hygiene Kits Distributed',
    img: 'https://github.com//Applied-Geotechnological-Solutions/Web_map_demo_data/blob/main/icons/WASH/1.hkit.png?raw=true',
    visible: true,
    inlegend: true,
    base_layer: false,
    layer_group: 'WASH',
  },

  {
    id: 11,
    name: 'New Bore Hole Drilled',
    img: 'https://github.com//Applied-Geotechnological-Solutions/Web_map_demo_data/blob/main/icons/WASH/2.nbore.png?raw=true',
    visible: true,
    inlegend: true,
    base_layer: false,
    layer_group: 'WASH',
  },

  {
    id: 12,
    name: 'Water Tower With Solar Powered Pump',
    img: 'https://github.com//Applied-Geotechnological-Solutions/Web_map_demo_data/blob/main/icons/WASH/3.wtower.png?raw=true',
    visible: true,
    inlegend: true,
    base_layer: false,
    layer_group: 'WASH',
  },

  {
    id: 13,
    name: 'New Latrines Built',
    img: 'https://github.com//Applied-Geotechnological-Solutions/Web_map_demo_data/blob/main/icons/WASH/4.nlat.png?raw=true',
    visible: true,
    inlegend: true,
    base_layer: false,
    layer_group: 'WASH',
  },

  {
    id: 14,
    name: 'Rehabilitated Borehole',
    img: 'https://github.com//Applied-Geotechnological-Solutions/Web_map_demo_data/blob/main/icons/WASH/5.rbore.png?raw=true',
    visible: true,
    inlegend: true,
    base_layer: false,
    layer_group: 'WASH',
  },

  {
    id: 15,
    name: 'Hygiene and Water Treatment Training',
    img: 'https://github.com//Applied-Geotechnological-Solutions/Web_map_demo_data/blob/main/icons/WASH/6.wtraining.png?raw=true',
    visible: true,
    inlegend: true,
    base_layer: false,
    layer_group: 'WASH',
  },

  {
    id: 16,
    name: 'WASH Assessment',
    img: 'https://github.com//Applied-Geotechnological-Solutions/Web_map_demo_data/blob/main/icons/WASH/7.wass.png?raw=true',
    visible: true,
    inlegend: true,
    base_layer: false,
    layer_group: 'WASH',
  },

  {
    id: 17,
    name: 'Emergency Water Trucking',
    img: 'https://github.com//Applied-Geotechnological-Solutions/Web_map_demo_data/blob/main/icons/Movement/2.wtruck.png?raw=true',
    visible: true,
    inlegend: true,
    base_layer: false,
    layer_group: 'WASH',
  },


  //   HEALTH
  {
    id: 18,
    name: 'Household Nutritional Training',
    img: 'https://github.com//Applied-Geotechnological-Solutions/Web_map_demo_data/blob/main/icons/Health/1.htraining.png?raw=true',
    visible: true,
    inlegend: true,
    base_layer: false,
    layer_group: 'Health',
  },

  {
    id: 19,
    name: 'Malaria Bednet Distribution',
    img: 'https://github.com//Applied-Geotechnological-Solutions/Web_map_demo_data/blob/main/icons/Health/2.malaria.png?raw=true',
    visible: true,
    inlegend: true,
    base_layer: false,
    layer_group: 'Health',
  },

  {
    id: 20,
    name: 'Pre and Post Natal Nutritional Workshop',
    img: 'https://github.com//Applied-Geotechnological-Solutions/Web_map_demo_data/blob/main/icons/Health/3.natal.png?raw=true',
    visible: true,
    inlegend: true,
    base_layer: false,
    layer_group: 'Health',
  },

  {
    id: 21,
    name: "Women's Health Consultation",
    img: 'https://github.com//Applied-Geotechnological-Solutions/Web_map_demo_data/blob/main/icons/Health/4.womenh.png?raw=true',
    visible: true,
    inlegend: true,
    base_layer: false,
    layer_group: 'Health',
  },

  {
    id: 22,
    name: 'Mobile Boat Clinic',
    img: 'https://github.com//Applied-Geotechnological-Solutions/Web_map_demo_data/blob/main/icons/Movement/1.boatc.png?raw=true',
    visible: true,
    inlegend: true,
    base_layer: false,
    layer_group: 'Health',
  },





// ASSETS

{
    id: 23,
    name: 'Country Office',
    img: 'https://github.com//Applied-Geotechnological-Solutions/Web_map_demo_data/blob/main/icons/Assets/1.countryo.png?raw=true',
    visible: true,
    inlegend: true,
    base_layer: false,
    layer_group: 'Assets',
  },

  {
    id: 24,
    name: 'Field Office',
    img: 'https://github.com//Applied-Geotechnological-Solutions/Web_map_demo_data/blob/main/icons/Assets/2.fieldo.png?raw=true',
    visible: true,
    inlegend: true,
    base_layer: false,
    layer_group: 'Assets',
  },

  {
    id: 25,
    name: 'Local Partner Office',
    img: 'https://github.com//Applied-Geotechnological-Solutions/Web_map_demo_data/blob/main/icons/Assets/3.partnero.png?raw=true',
    visible: true,
    inlegend: true,
    base_layer: false,
    layer_group: 'Assets',
  },

  {
    id: 26,
    name: 'Refrigerated Warehouse',
    img: 'https://github.com//Applied-Geotechnological-Solutions/Web_map_demo_data/blob/main/icons/Assets/4.refware.png?raw=true',
    visible: true,
    inlegend: true,
    base_layer: false,
    layer_group: 'Assets',
  },
  {
    id: 27,
    name: 'Warehouse',
    img: 'https://github.com//Applied-Geotechnological-Solutions/Web_map_demo_data/blob/main/icons/Assets/5.ware.png?raw=true',
    visible: true,
    inlegend: true,
    base_layer: false,
    layer_group: 'Assets',
  },

// Movement

  
  

])






const Baselayer_src = ref(layer_items.value[1].src)

const icon_size = ref(0.5)

const styles_items = ref([ // WHENADDLAYERS

{
    id: 0 ,
    name: 'base_layer',
    style: undefined
  },

  {
    id: 1 ,
    name: 'Food_Security',
    style: new Style({
            image: new Icon({
              src: layer_items.value[3].img,
              scale: [icon_size.value, icon_size.value],
              crossOrigin: undefined,
            }),
          })
  },

  {
    id: 2 ,
    name: 'Food_Security',
    style: new Style({
            image: new Icon({
              src: layer_items.value[4].img,
              scale: [icon_size.value, icon_size.value],
              crossOrigin: undefined,
            }),
          })
  },

  {
    id: 3 ,
    name: 'Food_Security',
    style: new Style({
            image: new Icon({
              src: layer_items.value[5].img,
              scale: [icon_size.value, icon_size.value],
              crossOrigin: undefined,
            }),
          })
  },

  {
    id: 4 ,
    name: 'Food_Security',
    style: new Style({
            image: new Icon({
              src: layer_items.value[6].img,
              scale: [icon_size.value, icon_size.value],
              crossOrigin: undefined,
            }),
          })
  },

  {
    id: 5 ,
    name: 'Food_Security',
    style: new Style({
            image: new Icon({
              src: layer_items.value[7].img,
              scale: [icon_size.value, icon_size.value],
              crossOrigin: undefined,
            }),
          })
  },

  {
    id: 6 ,
    name: 'Food_Security',
    style: new Style({
            image: new Icon({
              src: layer_items.value[8].img,
              scale: [icon_size.value, icon_size.value],
              crossOrigin: undefined,
            }),
          })
  },

  {
    id: 7 ,
    name: 'Food_Security',
    style: new Style({
            image: new Icon({
              src: layer_items.value[9].img,
              scale: [icon_size.value, icon_size.value],
              crossOrigin: undefined,
            }),
          })
  },

  {
    id: 8 ,
    name: 'WASH',
    style: new Style({
            image: new Icon({
              src: layer_items.value[10].img,
              scale: [icon_size.value, icon_size.value],
              crossOrigin: undefined,
            }),
          })
  },

  {
    id: 9 ,
    name: 'WASH',
    style: new Style({
            image: new Icon({
              src: layer_items.value[11].img,
              scale: [icon_size.value, icon_size.value],
              crossOrigin: undefined,
            }),
          })
  },

  {
    id: 10 ,
    name: 'WASH',
    style: new Style({
            image: new Icon({
              src: layer_items.value[12].img,
              scale: [icon_size.value, icon_size.value],
              crossOrigin: undefined,
            }),
          })
  },

  {
    id: 11 ,
    name: 'WASH',
    style: new Style({
            image: new Icon({
              src: layer_items.value[13].img,
              scale: [icon_size.value, icon_size.value],
              crossOrigin: undefined,
            }),
          })
  },

  {
    id: 12 ,
    name: 'WASH',
    style: new Style({
            image: new Icon({
              src: layer_items.value[14].img,
              scale: [icon_size.value, icon_size.value],
              crossOrigin: undefined,
            }),
          })
  },

  {
    id: 13 ,
    name: 'WASH',
    style: new Style({
            image: new Icon({
              src: layer_items.value[15].img,
              scale: [icon_size.value, icon_size.value],
              crossOrigin: undefined,
            }),
          })
  },

  {
    id: 14 ,
    name: 'WASH',
    style: new Style({
            image: new Icon({
              src: layer_items.value[16].img,
              scale: [icon_size.value, icon_size.value],
              crossOrigin: undefined,
            }),
          })
  },

  {
    id: 15 ,
    name: 'WASH',
    style: new Style({
            image: new Icon({
              src: layer_items.value[17].img,
              scale: [icon_size.value, icon_size.value],
              crossOrigin: undefined,
            }),
          })
  },

  {
    id: 16 ,
    name: 'Health',
    style: new Style({
            image: new Icon({
              src: layer_items.value[18].img,
              scale: [icon_size.value, icon_size.value],
              crossOrigin: undefined,
            }),
          })
  },

  {
    id: 17 ,
    name: 'Health',
    style: new Style({
            image: new Icon({
              src: layer_items.value[19].img,
              scale: [icon_size.value, icon_size.value],
              crossOrigin: undefined,
            }),
          })
  },

  {
    id: 18 ,
    name: 'Health',
    style: new Style({
            image: new Icon({
              src: layer_items.value[20].img,
              scale: [icon_size.value, icon_size.value],
              crossOrigin: undefined,
            }),
          })
  },

  {
    id: 19 ,
    name: 'Health',
    style: new Style({
            image: new Icon({
              src: layer_items.value[21].img,
              scale: [icon_size.value, icon_size.value],
              crossOrigin: undefined,
            }),
          })
  },

  {
    id: 19 ,
    name: 'Health',
    style: new Style({
            image: new Icon({
              src: layer_items.value[22].img,
              scale: [icon_size.value, icon_size.value],
              crossOrigin: undefined,
            }),
          })
  },


  {
    id: 20 ,
    name: 'Assets',
    style: new Style({
            image: new Icon({
              src: layer_items.value[23].img,
              scale: [icon_size.value, icon_size.value],
              crossOrigin: undefined,
            }),
          })
  },

  {
    id: 21 ,
    name: 'Assets',
    style: new Style({
            image: new Icon({
              src: layer_items.value[24].img,
              scale: [icon_size.value, icon_size.value],
              crossOrigin: undefined,
            }),
          })
  },

  {
    id: 22 ,
    name: 'Assets',
    style: new Style({
            image: new Icon({
              src: layer_items.value[25].img,
              scale: [icon_size.value, icon_size.value],
              crossOrigin: undefined,
            }),
          })
  },

  {
    id: 23 ,
    name: 'Assets',
    style: new Style({
            image: new Icon({
              src: layer_items.value[26].img,
              scale: [icon_size.value, icon_size.value],
              crossOrigin: undefined,
            }),
          })
  },

  {
    id: 24 ,
    name: 'Assets',
    style: new Style({
            image: new Icon({
              src: layer_items.value[27].img,
              scale: [icon_size.value, icon_size.value],
              crossOrigin: undefined,
            }),
          })
  },


  ]
)



//BASE LAYERS // WHENADDLAYERS

const Baselayer = ref(

    new TileLayer({
    name: 'Baselayer',
    visible: true,
    opacity: 0.9,
    source: Baselayer_src.value
    })
  )






// VECTOR LAYERS

// FOOD SECURITY

const Cash_Assistance = ref(new VectorLayer({
      name: 'Cash Assistance',
      group: 'Food Security',
      visible: layer_items.value[3].visible,
      source: new VectorSource({
        format: new GeoJSON(),
        url: 'https://raw.githubusercontent.com/Applied-Geotechnological-Solutions/Web_map_demo_data/main/Food/cash.json.geojson',
        crossOrigin: undefined,
      }),
      style: styles_items.value[1].style,
      zIndex:2,
  }))

const Fishing_Equipment_Distribution = ref(new VectorLayer({
      name: 'Fishing Equipment Distribution',
      group: 'Food Security',
      visible: layer_items.value[4].visible,
      source: new VectorSource({
        format: new GeoJSON(),
        url: 'https://raw.githubusercontent.com/Applied-Geotechnological-Solutions/Web_map_demo_data/main/Food/fish.json.geojson',
        crossOrigin: undefined,
      }),
      style: styles_items.value[2].style,
      zIndex:2,
  }))

const Food_Distribution = ref(new VectorLayer({
      name: 'Food Distribution',
      group: 'Food Security',
      visible: layer_items.value[5].visible,
      source: new VectorSource({
        format: new GeoJSON(),
        url: 'https://raw.githubusercontent.com/Syverpet/prototyping_data_april_2022/main/Access_to_food.json.geojson',
        crossOrigin: undefined,
      }),
      style: styles_items.value[3].style,
      zIndex:2,
  }))

  const Food_Security_Assessment = ref(new VectorLayer({
      name: 'Food Security Assessment',
      group: 'Food Security',
      visible: layer_items.value[6].visible,
      source: new VectorSource({
        format: new GeoJSON(),
        url: 'https://raw.githubusercontent.com/Applied-Geotechnological-Solutions/Web_map_demo_data/main/Food/fass.json.geojson',
        crossOrigin: undefined,
      }),
      style: styles_items.value[4].style,
      zIndex:2,
  }))

  const Food_Training = ref(new VectorLayer({
      name: 'Food Training',
      group: 'Food Security',
      visible: layer_items.value[7].visible,
      source: new VectorSource({
        format: new GeoJSON(),
        url: 'https://raw.githubusercontent.com/Applied-Geotechnological-Solutions/Web_map_demo_data/main/Food/ftraining.json.geojson',
        crossOrigin: undefined,
      }),
      style: styles_items.value[5].style,
      zIndex:2,
  }))

  const Farming_Equipment_Distribution = ref(new VectorLayer({
      name: 'Farming Equipment Distribution',
      group: 'Food Security',
      visible: layer_items.value[8].visible,
      source: new VectorSource({
        format: new GeoJSON(),
        url: 'https://raw.githubusercontent.com/Applied-Geotechnological-Solutions/Web_map_demo_data/main/Food/fequipment.json.geojson',
        crossOrigin: undefined,
      }),
      style: styles_items.value[6].style,
      zIndex:2,
  }))

  const Seed_Distribution = ref(new VectorLayer({
      name: 'Seed_Distribution',
      group: 'Food Security',
      visible: layer_items.value[9].visible,
      source: new VectorSource({
        format: new GeoJSON(),
        url: 'https://raw.githubusercontent.com/Applied-Geotechnological-Solutions/Web_map_demo_data/main/Food/seed.json.geojson',
        crossOrigin: undefined,
      }),
      style: styles_items.value[7].style,
      zIndex:2,
  }))



  // WASH

  const Hygiene_Kits_Distributed = ref(new VectorLayer({
      name: 'Hygiene Kits Distributed',
      group: 'WASH',
      visible: layer_items.value[10].visible,
      source: new VectorSource({
        format: new GeoJSON(),
        url: 'https://raw.githubusercontent.com/Applied-Geotechnological-Solutions/Web_map_demo_data/main/WASH/hkit.json.geojson',
        crossOrigin: undefined,
      }),
      style: styles_items.value[8].style,
      zIndex:2,
  }))

  const New_Bore_Hole_Drilled = ref(new VectorLayer({
      name: 'New Bore Hole Drilled',
      group: 'WASH',
      visible: layer_items.value[11].visible,
      source: new VectorSource({
        format: new GeoJSON(),
        url: 'https://raw.githubusercontent.com/Applied-Geotechnological-Solutions/Web_map_demo_data/main/WASH/nbore.json.geojson',
        crossOrigin: undefined,
      }),
      style: styles_items.value[9].style,
      zIndex:2,
  }))

  const Water_Tower_With_Solar_Powered_Pump = ref(new VectorLayer({
      name: 'Water Tower With Solar Powered Pump',
      group: 'WASH',
      visible: layer_items.value[12].visible,
      source: new VectorSource({
        format: new GeoJSON(),
        url: 'https://raw.githubusercontent.com/Applied-Geotechnological-Solutions/Web_map_demo_data/main/WASH/wtower.json.geojson',
        crossOrigin: undefined,
      }),
      style: styles_items.value[10].style,
      zIndex:2,
  }))

  const New_Latrines_Built = ref(new VectorLayer({
      name: 'New Latrines Built',
      group: 'WASH',
      visible: layer_items.value[13].visible,
      source: new VectorSource({
        format: new GeoJSON(),
        url: 'https://raw.githubusercontent.com/Applied-Geotechnological-Solutions/Web_map_demo_data/main/WASH/nlat.json.geojson',
        crossOrigin: undefined,
      }),
      style: styles_items.value[11].style,
      zIndex:2,
  }))

  const Rehabilitated_Borehole = ref(new VectorLayer({
      name: 'Rehabilitated Borehole',
      group: 'WASH',
      visible: layer_items.value[14].visible,
      source: new VectorSource({
        format: new GeoJSON(),
        url: 'https://raw.githubusercontent.com/Applied-Geotechnological-Solutions/Web_map_demo_data/main/WASH/rbore.json.geojson',
        crossOrigin: undefined,
      }),
      style: styles_items.value[12].style,
      zIndex:2,
  }))

  const Hygiene_and_Water_Treatment_Training = ref(new VectorLayer({
      name: 'Hygiene and Water Treatment Training',
      group: 'WASH',
      visible: layer_items.value[15].visible,
      source: new VectorSource({
        format: new GeoJSON(),
        url: 'https://raw.githubusercontent.com/Applied-Geotechnological-Solutions/Web_map_demo_data/main/WASH/wtraining.json.geojson',
        crossOrigin: undefined,
      }),
      style: styles_items.value[13].style,
      zIndex:2,
  }))

  const WASH_Assessment = ref(new VectorLayer({
      name: 'WASH Assessment',
      group: 'WASH',
      visible: layer_items.value[16].visible,
      source: new VectorSource({
        format: new GeoJSON(),
        url: 'https://raw.githubusercontent.com/Applied-Geotechnological-Solutions/Web_map_demo_data/main/WASH/wass.json.geojson',
        crossOrigin: undefined,
      }),
      style: styles_items.value[14].style,
      zIndex:2,
  }))

  //const Emergency_Water_Trucking = ref(new VectorLayer({
  //    name: 'Emergency Water Trucking',
  //    group: 'WASH',
  //    visible: layer_items.value[17].visible,
  //    source: new VectorSource({
  //      format: new GeoJSON(),
  //      url: 'https://raw.githubusercontent.com/Applied-Geotechnological-Solutions/Web_map_demo_data/main/WASH/wtruck.json.geojson',
  //      crossOrigin: undefined,
  //    }),
  //    style: styles_items.value[15].style,
  //    zIndex:2,
  //}))



  // HEALTH

    const Household_Nutritional_Training = ref(new VectorLayer({
      name: 'Household Nutritional Training',
      group: 'Health',
      visible: layer_items.value[18].visible,
      source: new VectorSource({
        format: new GeoJSON(),
        url: 'https://raw.githubusercontent.com/Applied-Geotechnological-Solutions/Web_map_demo_data/main/health/htraining.json.geojson',
        crossOrigin: undefined,
      }),
      style: styles_items.value[16].style,
      zIndex:2,
  }))

    const Malaria_Bednet_Distribution = ref(new VectorLayer({
      name: 'Malaria Bednet Distribution',
      group: 'Health',
      visible: layer_items.value[19].visible,
      source: new VectorSource({
        format: new GeoJSON(),
        url: 'https://raw.githubusercontent.com/Applied-Geotechnological-Solutions/Web_map_demo_data/main/health/malaria.json.geojson',
        crossOrigin: undefined,
      }),
      style: styles_items.value[17].style,
      zIndex:2,
  }))

    const Pre_and_Post_Natal_Nutritional_Workshop = ref(new VectorLayer({
      name: 'Pre and Post Natal Nutritional Workshop',
      group: 'Health',
      visible: layer_items.value[20].visible,
      source: new VectorSource({
        format: new GeoJSON(),
        url: 'https://raw.githubusercontent.com/Applied-Geotechnological-Solutions/Web_map_demo_data/main/health/natal.json.geojson',
        crossOrigin: undefined,
      }),
      style: styles_items.value[18].style,
      zIndex:2,
  }))

   const Women_Health_Consultation = ref(new VectorLayer({
      name: "Women's Health Consultation",
      group: 'Health',
      visible: layer_items.value[21].visible,
      source: new VectorSource({
        format: new GeoJSON(),
        url: 'https://raw.githubusercontent.com/Applied-Geotechnological-Solutions/Web_map_demo_data/main/health/womenh.json.geojson',
        crossOrigin: undefined,
      }),
      style: styles_items.value[19].style,
      zIndex:2,
  }))

  //const Boat_Clinic = ref(new VectorLayer({
  //    name: "Women's Health Consultation",
  //    group: 'Health',
  //    visible: layer_items.value[22].visible,
  //    source: new VectorSource({
  //      format: new GeoJSON(),
  //      url: 'https://raw.githubusercontent.com/Applied-Geotechnological-Solutions/Web_map_demo_data/main/health/womenh.json.geojson',
  //      crossOrigin: undefined,
  //    }),
  //    style: styles_items.value[20].style,
  //}))



  // ASSETS

    const Country_Office = ref(new VectorLayer({
      name: 'Country Office',
      group: 'Assets',
      visible: layer_items.value[23].visible,
      source: new VectorSource({
        format: new GeoJSON(),
        url: 'https://raw.githubusercontent.com/Applied-Geotechnological-Solutions/Web_map_demo_data/main/assets/countryo.json.geojson',
        crossOrigin: undefined,
      }),
      style: styles_items.value[21].style,
      zIndex:2,
  }))

  const Field_Office = ref(new VectorLayer({
      name: 'Field Office',
      group: 'Assets',
      visible: layer_items.value[24].visible,
      source: new VectorSource({
        format: new GeoJSON(),
        url: 'https://raw.githubusercontent.com/Applied-Geotechnological-Solutions/Web_map_demo_data/main/assets/fieldo.json.geojson',
        crossOrigin: undefined,
      }),
      style: styles_items.value[22].style,
      zIndex:2,
  }))

  const Local_Partner_Office = ref(new VectorLayer({
      name: 'Local Partner Office',
      group: 'Assets',
      visible: layer_items.value[25].visible,
      source: new VectorSource({
        format: new GeoJSON(),
        url: 'https://raw.githubusercontent.com/Applied-Geotechnological-Solutions/Web_map_demo_data/main/assets/partnero.json.geojson',
        crossOrigin: undefined,
      }),
      style: styles_items.value[23].style,
      zIndex:2,
  }))

  const Refrigerated_Warehouse = ref(new VectorLayer({
      name: 'Refrigerated Warehouse',
      group: 'Assets',
      visible: layer_items.value[26].visible,
      source: new VectorSource({
        format: new GeoJSON(),
        url: 'https://raw.githubusercontent.com/Applied-Geotechnological-Solutions/Web_map_demo_data/main/assets/refware.json.geojson',
        crossOrigin: undefined,
      }),
      style: styles_items.value[24].style,
      zIndex:2,
  }))

  const Warehouse = ref(new VectorLayer({
      name: 'Warehouse',
      group: 'Assets',
      visible: layer_items.value[27].visible,
      source: new VectorSource({
        format: new GeoJSON(),
        url: 'https://raw.githubusercontent.com/Applied-Geotechnological-Solutions/Web_map_demo_data/main/assets/ware.json.geojson',
        crossOrigin: undefined,
      }),
      style: styles_items.value[25].style,
      zIndex:2,
  }))


const map_layers = ref([                 // WHENADDLAYER

        // BASE
      Baselayer.value,

      // FOOD SECURITY

      Cash_Assistance.value,
      Fishing_Equipment_Distribution.value,
      Food_Distribution.value,
      Food_Security_Assessment.value,
      Food_Training.value,
      Farming_Equipment_Distribution.value,
      Seed_Distribution.value,

      // WASH
      Hygiene_Kits_Distributed.value,
      New_Bore_Hole_Drilled.value,
      Water_Tower_With_Solar_Powered_Pump.value,
      New_Latrines_Built.value,
      Rehabilitated_Borehole.value,
      Hygiene_and_Water_Treatment_Training.value,
      WASH_Assessment.value,
      //Emergency_Water_Trucking.value,

      // HEALTH
      Household_Nutritional_Training.value,
      Malaria_Bednet_Distribution.value,
      Pre_and_Post_Natal_Nutritional_Workshop.value,
      Women_Health_Consultation.value,
      //Boat_Clinic.value,


      // ASSETS
      Country_Office.value,
      Field_Office.value,
      Local_Partner_Office.value,
      Refrigerated_Warehouse.value,
      Warehouse.value,



      //Dark_Basemap.value,
      //Open_Street_Map.value,
      //Google_Satellite.value,

     ])




// MAP CONTROLS

const overviewMapControl = new OverviewMap({
  //className : 'overviewMap',
  layers: [
    new TileLayer({
      source: new OSM(),
    }),
  ],
});

const mousePositionControl = new MousePosition({
  className: 'mouse-position',
  coordinateFormat: createStringXY(4),
  projection: 'EPSG:4326',
  // comment the following two lines to have the mouse position
  // be placed within the map.
});

const zoomslider = new ZoomSlider({

      //className : 'zoomslider'
    }

  );

const scaleLine = new ScaleLine({
    //className : 'scaleLine'
  }
);

const toggle = ref(false)

// Initialize map
function initMap() {
  //OVERLAY  // WHENNEWOVERLAY
  overlay.value = new Overlay({ element: popupAttributes.value,
  autoPan: true,
  autoPanAnimation: {
  duration: 250 } })

  overlay2.value = new Overlay({ element: popupMessage1.value,
  autoPan: true,
  position: [-1815315.442215883, 1424799.1294523671],
  autoPanAnimation: {
  duration: 250 } })

  overlay3.value = new Overlay({ element: legend_overlay.value,
  autoPan: true,
  position: [],
  autoPanAnimation: {
  duration: 250 } })

  overlay4.value = new Overlay({ element: legend__toggle_overlay.value,
  autoPan: true,
  position: [],
  autoPanAnimation: {
  duration: 250 } })



  //MAP
  map.value = new Map({
    target: mapCom.value,
    controls: defaultControls().extend([overviewMapControl, mousePositionControl, zoomslider, scaleLine]),
    layers: map_layers.value,
      view: new View({ projection: 'EPSG:3857', // Projection coordinate system
      center: [-1815315.442215883, 1404799.1294523671], // The center of the map
      zoom: 11.5 // Map zoom level （ The default level when opening a page ）
      }),
      overlays: [overlay.value, overlay2.value, overlay3.value, overlay4.value] // WHENNEWOVERLAY
       // Bind a cover
      })

  map.value.controls.getArray()[0].getProperties()

  //map.value.addControl(pdfPrinter.value);

  mapClick() // Bind the click event after map initialization




 const Boat_Clinic_line_cords = [ [ -1840311.370282563846558, 1409272.195181151852012 ], [ -1840295.301365130813792, 1409276.685025676852092 ],
 [ -1840277.145575302885845, 1409277.967838525073603 ], [ -1840256.694225842598826, 1409273.905598033452407 ],
 [ -1840242.50349356350489, 1409267.491535242646933 ], [ -1840218.08708655484952, 1409256.801433710614219 ],
 [ -1840193.461992305936292, 1409243.545713220955804 ], [ -1840155.689601975260302, 1409218.530902163125575 ], [ -1840093.918179114349186, 1409182.612236408516765 ], [ -1839998.339423472294584, 1409105.643815017538145 ], [ -1839860.605845471378416, 1408960.687169650569558 ], [ -1839704.090415925718844, 1408825.993583544623107 ], [ -1839335.966125634731725, 1408604.071213982068002 ], [ -1839000.39704468823038, 1408501.448720353888348 ], [ -1838679.853444978129119, 1408450.137608096003532 ], [ -1838344.284364031394944, 1408537.366552306804806 ], [ -1838073.825701776193455, 1408732.349835646804422 ], [ -1837818.392520757392049, 1408958.121571757830679 ], [ -1837457.780971083557233, 1409147.976421793689951 ], [ -1837132.228877628222108, 1409214.682471585460007 ], [ -1836681.464440535753965, 1409214.682471585460007 ], [ -1836245.725484679918736, 1409122.320289182476699 ], [ -1836175.397885004524142, 1409106.712819487322122 ], [ -1836110.704840791877359, 1409107.568023092811927 ], [ -1836077.314882488921285, 1409117.830468283034861 ], [ -1836035.994809089228511, 1409129.37572341458872 ], [ -1836013.039212755626068, 1409147.762620595749468 ], [ -1835992.170488815754652, 1409165.721926641417667 ], [ -1835978.814505494665354, 1409184.536449530161917 ], [ -1835971.719139355001971, 1409171.28076169360429 ], [ -1835964.62377321603708, 1409155.031862194649875 ], [ -1835957.111032597487792, 1409135.789756006328389 ], [ -1835945.424547191476449, 1409110.561235902365297 ], [ -1835938.329181052045897, 1409085.760338980471715 ], [ -1835928.312193561578169, 1409062.242266437504441 ], [ -1835916.625708155333996, 1409036.586208802647889 ], [ -1835907.02609514282085, 1409013.495776110328734 ], [ -1835885.322622246108949, 1408984.418960796203464 ], [ -1835864.03652382758446, 1408955.769773887703195 ], [ -1835841.915676451986656, 1408931.824206290533766 ], [ -1835760.527653089025989, 1408883.077932640677318 ], [ -1835710.442715634359047, 1408854.428844681242481 ], [ -1835664.114148488733917, 1408830.910957559943199 ], [ -1835634.89793497370556, 1408808.675881616771221 ], [ -1835600.673227712977678, 1408778.744075222872198 ], [ -1835563.109524621628225, 1408751.80547557095997 ], [ -1835535.562809021677822, 1408727.432478148490191 ], [ -1835460.018028361024335, 1408659.872345501556993 ], [ -1835401.585601330501959, 1408564.091158173512667 ], [ -1835346.492170130368322, 1408466.599913506302983 ], [ -1835299.746228506090119, 1408362.267536693252623 ], [ -1835271.364763948367909, 1408240.831958557479084 ], [ -1835269.695266033057123, 1408081.769214739557356 ], [ -1835313.102211827179417, 1407965.465818456606939 ], [ -1835373.204136772779748, 1407842.321548061212525 ], [ -1835498.416480409447104, 1407679.84031486348249 ], [ -1835631.976313621737063, 1407495.125899562845007 ], [ -1835725.468196870293468, 1407395.927896734559909 ], [ -1835862.367025913437828, 1407245.421221844386309 ], [ -1835969.214892483316362, 1407070.971268114866689 ], [ -1836009.282842447282746, 1406828.111292634392157 ], [ -1835975.892884143861011, 1406557.889172082301229 ], [ -1835895.756984216161072, 1406301.351483251200989 ], [ -1835812.282088458538055, 1406061.918326982064173 ], [ -1835748.841167682548985, 1405808.805396341951564 ], [ -1835648.6712927732151, 1405596.73948326241225 ], [ -1835538.484430373180658, 1405343.630556508898735 ], [ -1835434.975559633225203, 1405114.466244778363034 ], [ -1835341.483676383970305, 1404919.506967026740313 ], [ -1835201.245851511135697, 1404680.085059935227036 ], [ -1835040.974051656201482, 1404437.244829741539434 ], [ -1834857.329280988778919, 1404266.231785468989983 ], [ -1834623.599572867155075, 1404081.538813577033579 ], [ -1834366.496893932810053, 1403917.36825573653914 ], [ -1834099.377227507997304, 1403763.4591891230084 ], [ -1833885.681494368007407, 1403643.752693478483707 ], [ -1833697.02822995511815, 1403566.798774664057419 ], [ -1833491.679986390750855, 1403539.437429766403511 ], [ -1833281.323249081615359, 1403578.76937104947865 ], [ -1833094.339482583571225, 1403583.899628133978695 ], [ -1832939.076176474336535, 1403571.929029662162066 ], [ -1832795.499355770647526, 1403566.798774664057419 ], [ -1832631.888560085324571, 1403563.378605160163715 ], [ -1832501.66772270295769, 1403553.118099037790671 ], [ -1832414.853831115178764, 1403582.189542338950559 ], [ -1832394.680731307249516, 1403585.467206864152104 ], [ -1832372.142509452532977, 1403593.447608974063769 ], [ -1832348.073914509033784, 1403597.865332500543445 ], [ -1832318.022952036233619, 1403601.713027723832056 ], [ -1832296.875978444004431, 1403608.553376030409709 ], [ -1832272.390009021619335, 1403613.541131004691124 ], [ -1832247.347540294518694, 1403620.951511383755133 ], [ -1832240.043486916227266, 1403604.135650902520865 ], [ -1832230.652561143506318, 1403577.84307472826913 ], [ -1832217.505265061743557, 1403545.137707857647911 ], [ -1832198.723413515836, 1403515.638780656503513 ], [ -1832184.323993997881189, 1403482.933482865570113 ], [ -1832164.916080734226853, 1403445.097987591987476 ], [ -1832143.629982315935194, 1403411.751489681890234 ], [ -1832117.961451870389283, 1403373.916085849748924 ], [ -1832077.89350190712139, 1403339.928391643799841 ], [ -1832041.581922251963988, 1403288.626285984646529 ], [ -1832000.261848852038383, 1403212.955843230709434 ], [ -1831962.698145760921761, 1403152.034865307155997 ], [ -1831888.196801297133788, 1403075.082231168402359 ], [ -1831834.981555251404643, 1403012.878998816478997 ], [ -1831696.621915533207357, 1402899.374470000620931 ], [ -1831543.862856296589598, 1402795.489352296805009 ], [ -1831416.14626578707248, 1402668.51915026595816 ], [ -1831332.253995550097898, 1402551.80964913032949 ], [ -1831227.075626895297319, 1402406.885307501070201 ], [ -1831096.854789513163269, 1402224.769003618042916 ], [ -1830987.920050549320877, 1402049.066312059061602 ], [ -1830963.816674399422482, 1402006.102674558525905 ], [ -1830930.009341617114842, 1401977.887782496633008 ], [ -1830904.966872890014201, 1401959.612241822527722 ], [ -1830879.611373303923756, 1401937.168610877357423 ], [ -1830843.612824507988989, 1401913.121882395585999 ], [ -1830815.753078048815951, 1401899.976345782866701 ], [ -1830799.475473376223817, 1401893.884513731347397 ], [ -1830804.014420832507312, 1401886.83081503235735 ], [ -1830811.840192309813574, 1401875.609025126323104 ], [ -1830820.292025505099446, 1401862.142882876330987 ], [ -1830827.491735264426097, 1401849.157680101692677 ], [ -1830837.821753614582121, 1401832.485329370945692 ], [ -1830847.369194816797972, 1401813.408324232557788 ], [ -1830862.551191481295973, 1401779.101808231091127 ], [ -1830873.820302408654243, 1401740.627258332446218 ], [ -1830888.219721927307546, 1401685.48049102188088 ], [ -1830891.976092235883698, 1401630.333826841088012 ], [ -1830892.602153954328969, 1401571.981073549482971 ], [ -1830888.845783645287156, 1401507.857301988406107 ], [ -1830885.089413336012512, 1401446.939848145935684 ], [ -1830878.202734436141327, 1401386.022520124912262 ], [ -1830867.559685226995498, 1401327.670250215101987 ], [ -1830846.27358680870384, 1401251.363609899301082 ], [ -1830786.797723581083119, 1401203.271290944423527 ], [ -1830734.208539254032075, 1401193.011606380576268 ], [ -1830661.585379944648594, 1401178.263316073687747 ], [ -1830585.83191204443574, 1401177.622086227871478 ], [ -1830513.208752734819427, 1401192.370376215549186 ], [ -1830443.089840298518538, 1401244.310064879478887 ], [ -1830394.257026279810816, 1401307.150798119604588 ], [ -1830344.172088825376704, 1401366.785495311254635 ], [ -1830282.191978725604713, 1401431.550195503979921 ], [ -1830216.455498315859586, 1401506.574827979318798 ], [ -1830185.152412406634539, 1401556.591355659067631 ], [ -1830162.614190552383661, 1401596.02752383146435 ], [ -1830141.015061274869367, 1401633.540025423280895 ], [ -1830121.920178870437667, 1401672.976296497741714 ], [ -1830113.155314815696329, 1401703.114620515843853 ], [ -1830105.016512479633093, 1401737.741669113514945 ], [ -1830102.825296465773135, 1401757.299557124264538 ], [ -1830104.390450761187822, 1401771.406894236104563 ], [ -1830105.016512479633093, 1401782.949265983421355 ], [ -1830096.721194714074954, 1401786.155481157591566 ], [ -1830086.860722651937976, 1401788.239521210314706 ],
 [ -1830075.278580866055563, 1401789.041075114393607 ], [ -1830060.87916134740226, 1401788.399831988383085 ], [ -1830050.079596708994359, 1401782.949265981093049 ], [ -1830043.818979527335614, 1401773.971865344326943 ], [ -1830040.532155506778508, 1401762.269186472287402 ], [ -1830040.532155506778508, 1401753.131481543416157 ], [ -1830043.349433238850906, 1401743.192226763814688 ], [ -1830049.766565850004554, 1401729.726144987391308 ], [ -1830054.30551330675371, 1401718.344104471150786 ], [ -1830062.444315643515438, 1401703.114620513515547 ], [ -1830071.365695127286017, 1401687.564523956505582 ], [ -1830079.661012893309817, 1401671.212884875945747 ], [ -1830090.147546672727913, 1401653.899394555017352 ], [ -1830099.851503304438666, 1401637.066844265442342 ], [ -1830109.242429077159613, 1401621.356472664512694 ], [ -1830121.763663440709934, 1401604.684250731952488 ], [ -1830128.963373200269416, 1401588.653276992961764 ], [ -1830143.675823577679694, 1401572.622311968356371 ], [ -1830154.631903645582497, 1401555.950117587344721 ], [ -1830165.274952854728326, 1401540.560408062068745 ], [ -1830174.039816909236833, 1401523.24699444626458 ], [ -1830186.248020414030179, 1401510.742868710076436 ], [ -1830201.899563368177041, 1401496.315037912223488 ], [ -1830223.18566178670153, 1401474.833614023169503 ], [ -1830267.323012918233871, 1401427.702783597866073 ], [ -1830301.75640741805546, 1401390.190543820383027 ], [ -1830334.311616763938218, 1401356.525753872469068 ], [ -1830367.492887827800587, 1401317.731138821924105 ], [ -1830394.100510850315914, 1401286.310744196875021 ], [ -1830418.516917859436944, 1401252.646072811214253 ], [ -1830471.106102186953649, 1401194.935296964133158 ], [ -1830489.887953732395545, 1401173.774707443313673 ], [ -1830511.800113868433982, 1401162.873803613008931 ], [ -1830539.659860327607021, 1401154.858435720205307 ], [ -1830559.537319880211726, 1401152.934747750638053 ], [ -1830588.33615891658701, 1401155.17905039456673 ], [ -1830612.126504207262769, 1401157.102738510584459 ], [ -1830644.055651834933087, 1401159.347041471861303 ], [ -1830670.663274857681245, 1401163.194418374681845 ], [ -1830695.39271272555925, 1401167.041795778553933 ], [ -1830763.946470867143944, 1401175.05716698220931 ], [ -1830843.769339935155585, 1401183.713770327391103 ], [ -1830890.723968798993155, 1401193.332221467979252 ], [ -1830943.31315312650986, 1401213.530979074537754 ], [ -1830962.721066390164196, 1401252.00484134699218 ], [ -1830965.538344121770933, 1401299.45600726432167 ], [ -1830966.164405840681866, 1401360.693773401435465 ], [ -1830966.164405840681866, 1401437.000696561997756 ], [ -1830965.538344122469425, 1401512.025342902168632 ], [ -1830968.668652713298798, 1401595.386284902924672 ], [ -1830976.181393330916762, 1401671.693815306993201 ], [ -1830991.832936284597963, 1401753.131481543416157 ], [ -1831018.127528448589146, 1401819.820761554641649 ], [ -1831049.430614357348531, 1401900.617591334972531 ], [ -1831074.473083085147664, 1401977.567158877849579 ], [ -1831128.31439084908925, 1402062.211915144929662 ], [ -1831160.869600194273517, 1402137.879402997903526 ], [ -1831204.693920467514545, 1402198.157032307004556 ], [ -1831259.787351667415351, 1402260.999798357486725 ], [ -1831312.376535994699225, 1402314.86513303569518 ], [ -1831361.209350013406947, 1402346.927878983784467 ], [ -1831420.059151522349566, 1402370.013077662093565 ], [ -1831443.379950524540618, 1402375.944694131612778 ], [ -1831460.440132345072925, 1402376.906577996211126 ], [ -1831473.430912997806445, 1402376.746264015324414 ], [ -1831488.456394233508036, 1402375.784380157012492 ],
 [ -1831499.255958872614428, 1402373.860612533753738 ], [ -1831503.638390900101513, 1402372.097158988239244 ], [ -1831502.855813752161339, 1402377.547833923483267 ], [ -1831499.882020590826869, 1402385.723848214838654 ], [ -1831496.282165710814297, 1402394.060178846353665 ], [ -1831489.552002240670845, 1402409.951315644429997 ], [ -1831482.665323340799659, 1402432.074677989352494 ], [ -1831475.152582722250372, 1402454.198056941386312 ], [ -1831467.95287296269089, 1402480.489630334544927 ], [ -1831461.066194062819704, 1402508.384374090237543 ], [ -1831452.301330008078367, 1402548.783704957226291 ], [ -1831454.80557688116096, 1402597.519479386042804 ], [ -1831459.188008908648044, 1402657.798048724420369 ], [ -1831474.839551863493398, 1402702.045162137364969 ], [ -1831496.751711999764666, 1402732.184538299683481 ], [ -1831528.054797909688205, 1402757.835095484973863 ], [ -1831568.74880959186703, 1402781.561880761524662 ], [ -1831618.833747047232464, 1402807.853746221633628 ], [ -1831675.179301684955135, 1402834.786901017883793 ], [ -1831739.037596940761432, 1402863.002613399643451 ], [ -1831792.252842986723408, 1402888.012017417466268 ], [ -1831873.01480463379994, 1402930.976940549677238 ], [ -1831989.462284217588603, 1402992.538730657659471 ], [ -1832119.68312160205096, 1403038.710157654481009 ], [ -1832321.27499486040324, 1403093.859456958482042 ], [ -1832614.27187897474505, 1403166.964501213748008 ], [ -1832849.671085015404969, 1403218.266394985141233 ], [ -1833062.532069200649858, 1403265.720726269530132 ], [ -1833277.897300259210169, 1403314.457686623558402 ], [ -1833394.344779843464494, 1403315.74023930192925 ], [ -1833423.769680598285049, 1403300.990886884508654 ], [ -1833448.812149325851351, 1403282.393887842074037 ], [ -1833466.967939153779298, 1403253.536498780595139 ], [ -1833476.358864926965907, 1403229.809333395911381 ], [ -1833481.993420390412211, 1403199.028174351667985 ], [ -1833489.506161008728668, 1403213.777475713985041 ], [ -1833505.157703963574022, 1403242.634825825225562 ], [ -1833523.939555509015918, 1403275.179537925869226 ], [ -1833549.764601384289563, 1403316.221196569968015 ], [ -1833584.197995884576812, 1403362.393130899174139 ], [ -1833624.892007567454129, 1403407.28258088696748 ], [ -1833673.724821586627513, 1403440.629073738120496 ], [ -1833724.435820760205388, 1403468.204086729558185 ], [ -1833782.033498833887279, 1403492.572724350728095 ], [ -1833827.73600426223129, 1403523.354190165642649 ], [ -1833889.090052645187825, 1403556.059532846324146 ], [ -1833956.078656492056325, 1403591.330041147070006 ], [ -1834016.180581438587978, 1403627.883157960372046 ], [ -1834125.115320404060185, 1403686.881266776239499 ], [ -1834376.792131117545068, 1403821.551305836532265 ], [ -1834589.653115303255618, 1403960.069702879758552 ], [ -1834845.086296326247975, 1404134.501203985419124 ], [ -1835025.392071166308597, 1404288.412210334092379 ], [ -1835165.62989604123868, 1404439.758818004047498 ], [ -1835280.825252189068124, 1404639.845358300721273 ], [ -1835418.558830191614106, 1404909.194772420451045 ], [ -1835508.711717610945925, 1405104.153981840237975 ], [ -1835618.898580013075843, 1405373.507646777899936 ], [ -1835699.034479941939935, 1405571.035235384246334 ], [ -1835816.73408296238631, 1405873.742426162352785 ], [ -1835894.365736018167809, 1406094.36149457260035 ], [ -1835974.5016359470319, 1406366.289601034251973 ], [ -1836022.082326529547572, 1406592.043147763470188 ], [ -1836049.629042130196467, 1406856.279615463223308 ], [ -1836024.586573402630165, 1407064.078895071987063 ], [ -1835979.510129692731425, 1407220.570680785225704 ], [ -1835866.819020417518914, 1407353.974172744434327 ], [ -1835772.909762688679621, 1407439.275761605938897 ],
 [ -1835680.252628395799547, 1407517.522549688117579 ], [ -1835595.734296439914033, 1407608.596942603820935 ], [ -1835531.876001184573397, 1407690.692412058822811 ], [ -1835458.626780155580491, 1407803.574057295219973 ], [ -1835389.75999115454033, 1407922.228527185274288 ], [ -1835351.570226344512776, 1408027.414512243121862 ], [ -1835327.153819334926084, 1408135.166399879381061 ], [ -1835324.649572462076321, 1408243.5600668231491 ], [ -1835344.05748572642915, 1408352.595520210918039 ], [ -1835381.621188818011433, 1408438.54139833082445 ], [ -1835434.83643486420624, 1408535.391159585909918 ], [ -1835496.816544965142384, 1408612.999477076577023 ], [ -1835574.448198020923883, 1408686.118244955781847 ], [ -1835645.819233894813806, 1408741.278137771645561 ], [ -1835718.44239320512861, 1408795.796738365897909 ], [ -1835797.952231415780261, 1408841.335882383165881 ], [ -1835882.470563371898606, 1408891.364882633090019 ], [ -1835997.039857801282778, 1408947.166560467798263 ], [ -1836119.747954567195848, 1409000.402742835460231 ], [ -1836349.512605144409463, 1409060.053037832956761 ], [ -1836577.399070566752926, 1409111.365216585574672 ], [ -1836809.041906298371032, 1409140.8697600052692 ], [ -1836986.84343426534906, 1409143.435373878106475 ], [ -1837189.68743095966056, 1409107.516800065292045 ], [ -1837385.018687035888433, 1409040.810993938241154 ], [ -1837607.896658712998033, 1408944.600963757373393 ], [ -1837798.219421043992043, 1408820.169792570406571 ], [ -1837973.516702137887478, 1408703.43587455782108 ], [ -1838041.131367702968419, 1408617.489220731658861 ], [ -1838061.165342685300857, 1408517.432239322923124 ], [ -1838069.930206740042195, 1408440.465562661876902 ], [ -1838071.182330176234245, 1408337.843641061335802 ], [ -1838081.199317667400464, 1408227.525475482922047 ], [ -1838089.964181722374633, 1408119.773248861311004 ], [ -1838115.006650449708104, 1408047.938650827854872 ], [ -1838155.07460041414015, 1407961.993916258681566 ], [ -1838197.646797251421958, 1407885.028695838991553 ], [ -1838241.471117524895817, 1407820.891166262794286 ], [ -1838294.060301852645352, 1407769.581243467517197 ], [ -1838370.439831472700462, 1407727.250624655745924 ], [ -1838384.839250990655273, 1407716.347293328028172 ], [ -1838392.35199160897173, 1407700.312989908503368 ], [ -1838395.482300200033933, 1407689.40966858365573 ], [ -1838395.169269340811297, 1407678.185665566008538 ], [ -1838402.682009959360585, 1407682.675266258651391 ], [ -1838420.524768927600235, 1407694.540642820531502 ], [ -1838435.550250164465979, 1407703.840535909635946 ], [ -1838453.079978273948655, 1407713.14043194311671 ], [ -1838469.98364466545172, 1407722.119644702645019 ], [ -1838491.269743083510548, 1407732.702291836496443 ], [ -1838509.7385637704283, 1407743.926315690623596 ], [ -1838530.711631329730153, 1407761.243389481445774 ], [ -1838551.058637171285227, 1407779.843220854876563 ], [ -1838572.03170473058708, 1407802.611995986429974 ], [ -1838591.752648853929713, 1407826.342850828543305 ], [ -1838617.734210158698261, 1407872.201178715564311 ], [ -1838653.497985811205581, 1407942.111214988864958 ], [ -1838696.070182648487389, 1408006.890388006111607 ], [ -1838738.642379485769197, 1408070.386943832039833 ], [ -1838788.10125522268936, 1408122.980154658434913 ], [ -1838836.934069241629913, 1408161.463051538681611 ], [ -1838893.905685597565025, 1408191.608022663975134 ], [ -1838930.843326970236376, 1408224.639249971369281 ], [ -1838970.911276934668422, 1408269.856815579347312 ], [ -1839026.004708135966212, 1408318.281377970473841 ], [ -1839085.167540505062789, 1408361.574928540503606 ], [ -1839146.20855802949518, 1408417.375599019462243 ], [ -1839220.709902493981645, 1408463.555544468108565 ],
 [ -1839311.488851632224396, 1408509.735562574584037 ], [ -1839401.641739052254707, 1408552.067309668753296 ], [ -1839515.584971763426438, 1408615.565044795628637 ], [ -1839624.519710728898644, 1408686.118244954617694 ], [ -1839706.533795811934397, 1408753.464639700483531 ], [ -1839791.678189486265182, 1408819.528396148001775 ], [ -1839869.935904260026291, 1408887.516494968906045 ], [ -1839963.845161989331245, 1408975.388139090267941 ], [ -1840025.19921037228778, 1409056.846029639942572 ], [ -1840081.54476500954479, 1409108.158202783903107 ], [ -1840125.99514700146392, 1409142.152566913282499 ], [ -1840162.932788374833763, 1409181.919608905911446 ], [ -1840193.609812566079199, 1409218.479678943520412 ], [ -1840218.495765864383429, 1409245.739409940550104 ], [ -1840239.312317994423211, 1409263.538424436701462 ], [ -1840254.337799230590463, 1409270.433541061356664 ], [ -1840268.737218749243766, 1409272.678463104180992 ], [ -1840279.693298817612231, 1409273.480221018427983 ], [ -1840290.805894315708429, 1409273.800924189155921 ], [ -1840298.670794650679454, 1409274.041451570577919 ], [ -1840305.087927262065932, 1409273.319869433064014 ],
 [ -1840311.370554426684976, 1409272.194902872201055 ] ]

 const Water_T_line_cords = [ [ -1827666.645244958112016, 1420361.235884354682639 ], [ -1827544.7718971518334, 1420433.100744455587119 ], [ -1827437.924030581954867, 1420561.431293039349839 ], [ -1827148.266142303589731, 1420409.145771378651261 ], [ -1826927.892417503753677, 1420341.558632168220356 ], [ -1826538.064654315588996, 1420330.436712903203443 ], [ -1826101.490949503844604, 1420246.594689023448154 ], [ -1825926.193668412743136, 1420192.696372592821717 ], [ -1825646.55276762531139, 1419977.959626286290586 ], [ -1825506.314942752476782, 1419911.22900770814158 ], [ -1825410.318812631303445, 1419893.263098063878715 ], [ -1824698.277951819356531, 1420007.047379672992975 ], [ -1824270.886485541705042, 1420164.891724557615817 ], [ -1823661.519746511708945, 1420170.024888351326808 ], [ -1823498.743699784157798, 1420158.047507575014606 ], [ -1823008.746061687357724, 1420062.228638738160953 ], [ -1822885.620590445119888, 1420006.191857100231573 ], [ -1822697.8020749904681, 1419972.826496350811794 ], [ -1822627.265788074815646, 1419968.976649494376034 ], [ -1822520.835295984055847, 1420001.058722191024572 ], [ -1822439.447272620629519, 1420030.146498650778085 ], [ -1822344.285891456995159, 1420036.56292384210974 ], [ -1822232.012156662996858, 1420054.956683888565749 ], [ -1822162.727993183769286, 1420069.928357621422037 ], [ -1822054.210628698579967, 1420063.511925078229979 ], [ -1821934.006778807612136, 1420052.390112021705136 ], [ -1821884.339215831831098, 1420042.551588628208265 ], [ -1821744.936139917233959, 1420047.256968965521082 ], [ -1821632.662405123701319, 1420045.545921483077109 ], [ -1821581.325344232609496, 1420016.885891072219238 ], [ -1821553.361254153773189, 1420009.827828210312873 ], [ -1821577.151599444681779, 1420059.448185199638829 ], [ -1821588.838084850693122, 1420100.299490837613121 ], [ -1821579.02978459908627, 1420115.485069325892255 ], [ -1821512.041180753614753, 1420152.058819036232308 ], [ -1821482.198905520141125, 1420151.844937324756756 ], [ -1821456.11300059617497, 1420158.475271090632305 ], [ -1821503.902378417318687, 1420247.877983486512676 ], [ -1821553.15256691374816, 1420332.575543201295659 ], [ -1821589.46414656820707, 1420388.612952998839319 ], [ -1821618.680360083002597, 1420446.361541600199416 ], [ -1821634.957964756060392, 1420522.504300192696974 ], [ -1821657.078812131891027, 1420607.202658911701292 ], [ -1821692.973017307231203, 1420703.451092978008091 ], [ -1821766.013551095500588, 1420800.127618263009936 ], [ -1821824.863352604676038, 1420885.254559059860185 ], [ -1821846.984199980041012, 1420923.326487538870424 ], [ -1821438.374585246900097, 1421006.742684723809361 ], [ -1820874.292977166594937, 1421116.253542504040524 ], [ -1820775.010023024864495, 1421272.980980299413204 ], [ -1820712.403851206647232, 1421421.848765455186367 ], [ -1820593.869499230524525, 1421621.195900695631281 ], [ -1820483.68263682955876, 1421850.489574021659791 ], [ -1820413.563724393257871, 1421978.826374250696972 ], [ -1820335.932071338407695, 1422020.749851713888347 ], [ -1820291.690376586979255, 1422024.172179072862491 ], [ -1820203.206987083656713, 1421980.537535412469879 ], [ -1820108.045605919789523, 1421943.747592610307038 ], [ -1820032.918199737556279, 1421894.124022556934506 ], [ -1819997.858743519289419, 1421830.811314729042351 ], [ -1819954.451797725865617, 1421755.520706445677206 ], [ -1819938.591567531926557, 1421712.314252206822857 ], [ -1819936.087320659542456, 1421666.113362246192992 ], [ -1819921.061839422909543, 1421609.645707508316264 ], [ -1819910.62747745309025, 1421583.550843488890678 ], [ -1819868.472655095858499, 1421541.627996207913384 ], [ -1819831.743700962048024, 1421476.177141144871712 ], [ -1819800.857989531941712, 1421412.865343030542135 ], [ -1819784.997759337536991, 1421353.403712161350995 ], [ -1819774.563397367950529, 1421331.586885268101469 ], [ -1819683.575760991778225, 1421333.511898747412488 ], [ -1819585.075383997289464, 1421318.967355612665415 ], [ -1819465.70628306386061, 1421285.600490008946508 ], [ -1819359.693165451055393, 1421253.08922187006101 ], [ -1819327.972705063410103, 1421339.500830386532471 ], [ -1819274.548771778121591, 1421517.458218854153529 ], [ -1819239.906690038740635, 1421636.810058037051931 ], [ -1819292.49587436649017, 1421460.563070024596527 ], [ -1819358.441042015096173, 1421252.661442267941311 ], [ -1819507.026356463786215, 1421296.294994047377259 ], [ -1819634.742946973303333, 1421328.806310466490686 ], [ -1819738.251817713258788, 1421330.517433390486985 ], [ -1819774.146022888598964, 1421331.372994888108224 ], [ -1819806.701232234714553, 1421233.839146015932783 ], [ -1819835.91744574951008, 1421163.683421804336831 ], [ -1819818.387717640493065, 1421085.827877097763121 ], [ -1819810.040228064870462, 1420966.05052276304923 ], [ -1819819.222466598032042, 1420879.640023260610178 ], [ -1819886.837132161715999, 1420770.985600890824571 ], [ -1819956.121295640943572, 1420690.564478628570214 ], [ -1820103.871861132560298, 1420616.987901839660481 ], [ -1820232.423200599616393, 1420548.544741564197466 ], [ -1820398.53824315732345, 1420469.83530619321391 ], [ -1820446.953682696679607, 1420459.568873791955411 ], [ -1820489.525879533262923, 1420465.557625586399809 ], [ -1820551.297302393941209, 1420418.503180387429893 ], [ -1820620.581465873401612, 1420357.760281776310876 ], [ -1820689.030880394857377, 1420309.850479315500706 ], [ -1820751.637052213540301, 1420281.404071408789605 ], [ -1820785.235697755822912, 1420271.137705455766991 ], [ -1820797.548244879813865, 1420252.102161494549364 ], [ -1820833.025075577199459, 1420270.068292542826384 ], [ -1820880.188391680829227, 1420285.0400768853724 ], [ -1820976.184521801769733, 1420334.660902890609577 ], [ -1821066.754783699288964, 1420392.409384470200166 ], [ -1821166.92465860885568, 1420453.580123228253797 ], [ -1821270.433529348112643, 1420507.478922645328566 ], [ -1821333.874450124101713, 1420528.439593813149258 ], [ -1821448.44374455162324, 1420614.635165535379201 ], [ -1821577.621145737357438, 1420670.031461745500565 ], [ -1821658.591794622130692, 1420717.941857004771009 ], [ -1821713.685225822264329, 1420735.480503558879718 ], [ -1821751.248928913613781, 1420781.252143130404875 ], [ -1821797.57749605900608, 1420851.834719070699066 ], [ -1821847.453746274113655, 1420922.845240939874202 ], [ -1822214.534600369399413, 1420846.915321863722056 ], [ -1822642.343441128497943, 1420759.649631372187287 ], [ -1822959.130670529324561, 1420697.622706527356058 ], [ -1823338.106697269948199, 1420546.619780007749796 ], [ -1823682.023267791606486, 1420402.461909634526819 ], [ -1823978.776522211264819, 1420283.542898103129119 ], [ -1824291.390006824396551, 1420157.352391873719171 ], [ -1824617.776849237270653, 1420027.312577971024439 ], [ -1824681.635144492378458, 1420007.207790155196562 ], [ -1824937.485699989832938, 1419963.576170727843419 ], [ -1825247.594937730580568, 1419912.244937449460849 ], [ -1825342.338944415561855, 1419894.279027177719399 ], [ -1825415.796852682251483, 1419890.429190704133362 ], [ -1825500.941246355185285, 1419911.389417660655454 ], [ -1825638.674824355170131, 1419970.420341999735683 ], [ -1825808.963611701270565, 1420108.58739360421896 ], [ -1825923.324218889232725, 1420189.862436093622819 ], [ -1826093.61300623556599, 1420243.33298250916414 ], [ -1826377.010277332970873, 1420304.07564237806946 ], [ -1826526.847715218085796, 1420328.458295012824237 ], [ -1826810.036299076396972, 1420337.013616568176076 ], [ -1826937.335515107261017, 1420342.78846003790386 ], [ -1827075.90384206478484, 1420372.304344551404938 ], [ -1827221.567535162670538, 1420444.169231975451112 ], [ -1827435.680642781080678, 1420558.811205345671624 ], [ -1827584.474644469562918, 1420634.526487562339753 ], [ -1827792.327134906314313, 1420749.169210651889443 ], [ -1827962.615922252414748, 1420847.556982322363183 ], [ -1828101.184249210637063, 1420921.989691450726241 ], [ -1828224.727094931993634, 1421056.311269001103938 ], [ -1828167.129416859243065, 1420933.967386565404013 ], [ -1828130.40046272543259, 1420866.379028717055917 ], [ -1828056.107805501203984, 1420695.269963702652603 ], [ -1827965.12016912503168, 1420533.572821705136448 ], [ -1827874.132532749092206, 1420341.932927598012611 ], [ -1827838.238327573053539, 1420270.923822867684066 ], [ -1827663.775795439491048, 1420362.465713054640219 ] ]


  const Boat_Clinic_route = new LineString(Boat_Clinic_line_cords)
  const Water_T_route = new LineString(Water_T_line_cords)


  const Boat_Clinic_feature = new Feature({
              geometry: Boat_Clinic_route,
              name: 'Mobile Boat Clinic'
          })
  const Water_T_feature = new Feature({
              geometry: Water_T_route,
              name: 'Emergency Water Truck'
          })


  const Boat_Clinic_Lines = new VectorLayer({
      source: new VectorSource({
          features: [Boat_Clinic_feature]
      }),
  })
  const Water_T_Lines = new VectorLayer({
      source: new VectorSource({
          features: [Water_T_feature]
      }),
  })


var Boat_Clinic_position = new Point(Boat_Clinic_route.getFirstCoordinate())

var Water_T_position = new Point(Water_T_route.getFirstCoordinate())

   //console.log('Boat_Clinic_route', Boat_Clinic_route.getFirstCoordinate())

  //console.log('layerLines', layerLines)


  const Boat_Clinic_point_features = new Feature({
      type: 'Mobile Boat Clinic',
      geometry: Boat_Clinic_position,
    })
  const Boat_Clinic_style = new Style({
      image: new Icon({
      src: 'https://github.com//Applied-Geotechnological-Solutions/Web_map_demo_data/blob/main/icons/Movement/1.boatc.png?raw=true',
      crossOrigin: undefined,
      scale: [icon_size.value, icon_size.value],
            }),
          })

  const Water_T_point_features = new Feature({
      type: 'Emergency Water Trucking',
      geometry: Water_T_position,
    })
  const Water_T_style = new Style({
      image: new Icon({
      src: 'https://github.com//Applied-Geotechnological-Solutions/Web_map_demo_data/blob/main/icons/Movement/2.wtruck.png?raw=true',
      crossOrigin: undefined,
      scale: [icon_size.value, icon_size.value],
            }),
          })

// LAYERS Movement

  const Boat_Clinic_Point = new VectorLayer({
      source: new VectorSource({
          features: [Boat_Clinic_point_features]
      }),
      style: Boat_Clinic_style,
      zIndex:2,
  })
  const Boat_Clinic_Stops = new VectorLayer({
      source: new VectorSource({
        format: new GeoJSON(),
        url: 'https://raw.githubusercontent.com/Applied-Geotechnological-Solutions/Web_map_demo_data/main/test_river_stops.json.geojson',
        crossOrigin: undefined,
      }),
  })

  const Water_T_Point = new VectorLayer({
      source: new VectorSource({
          features: [Water_T_point_features]
      }),
      style: Water_T_style,
      zIndex:2,
  })
  const Water_T_Stops = new VectorLayer({
      source: new VectorSource({
        format: new GeoJSON(),
        url: 'https://raw.githubusercontent.com/Applied-Geotechnological-Solutions/Web_map_demo_data/main/water_truck_stops.json.geojson',
        crossOrigin: undefined,
      }),
  })
  

  // ADD EXTRA LAYERS
  map.value.addLayer(Boat_Clinic_Lines);
  map.value.addLayer(Boat_Clinic_Point);
  map.value.addLayer(Boat_Clinic_Stops);

  map.value.addLayer(Water_T_Lines);
  map.value.addLayer(Water_T_Point);
  map.value.addLayer(Water_T_Stops);
  

    var distance = 0;
    var distance2 = 0;
    var distance3 = 0.0003;
    var lastTime;

    function moveFeature(event) {

      // style called again to enable scaling of icon
      

      const speed = 10;
      const time = event.frameState.time;
      const elapsedTime = time - lastTime;
      distance2 = distance2 + distance3
      distance = ((distance2 + (speed * elapsedTime) / 1000000) % 2)
      //distance3 = distance
      lastTime = time;

      //console.log('distance1', Number.isNaN(distance + (speed * elapsedTime)))

      const Boat_Clinic_currentCoordinate = Boat_Clinic_route.getCoordinateAt(
        distance > 1 ? 2 - distance : distance
      );

      const Water_T_currentCoordinate = Water_T_route.getCoordinateAt(
        distance > 1 ? 2 - distance : distance
      );
      //console.log('currentCoordinate', currentCoordinate)

      //console.log('speed', speed)
      //console.log('distance2', distance2)
      //console.log('elapsedTime', elapsedTime)

      //console.log((distance > 1 ? 2 - distance : distance))

      Boat_Clinic_position.setCoordinates(Boat_Clinic_currentCoordinate);

      Water_T_position.setCoordinates(Water_T_currentCoordinate);

      var vectorContext = getVectorContext(event);

      vectorContext.setStyle(Boat_Clinic_Point.getStyle());
      vectorContext.drawGeometry(Boat_Clinic_position);

      vectorContext.setStyle(Water_T_Point.getStyle());
      vectorContext.drawGeometry(Water_T_position);
      
      

      // tell OpenLayers to continue the postrender animation
      map.value.render();
    }

    Boat_Clinic_Point.on('postrender', moveFeature);
    Water_T_Point.on('postrender', moveFeature);

}






function closePopup1() {
    overlay.value.setPosition(undefined) // setPosition Pass in undefined Will hide pop-up elements

    selected.value.setStyle(undefined);

    currentCoordinate.value = '' // Empty the pop-up window
}

function closePopup2() {

    overlay2.value.setPosition(undefined) // setPosition Pass in undefined Will hide pop-up elements
}

function change_layer_visibility(layer_name, group_visibility) {

if (group_visibility == true) {

  console.log(layer_name)
  map.value.getLayers().forEach(function(layer) {

    console.log('layer_visibility_before:', layer.getVisible())
    if (layer.get('name') == layer_name) {
        layer.setVisible(!layer.getVisible())
        }
    console.log('layer_visibility_after:', layer.getVisible())
    })
}
}

function change_baselayer() {
  console.log('change_baselayer')
  map.value.getLayers().forEach(function(layer) {
    console.log(layer.get('name'))
  if (layer.get('name') == 'Baselayer') {
    layer.setSource(Baselayer_src.value)
    console.log(Baselayer_src.value)
  }
})}

function change_layer_group_visibility(layer_group, group_visibility) {

  console.log(layer_group)

  map.value.getLayers().forEach(function(layer) {

    console.log(layer)

  if (layer.get('group') == layer_group) {

    layer.setVisible(group_visibility)

  }
    var item_array = [...Array(layer_items.value.length).keys()]

    item_array.forEach(function(n)  {


    console.log(item_array)

    if (layer_items.value[n].layer_group == layer_group) {

    layer_items.value[n].visible = group_visibility

        }
    })

})
}

function change_icon_scale(size) {

  var layer_array = [...Array(map_layers.value.length).keys()] // create array 0, 1,2,3 ... the lenght of map_layers

  console.log(layer_array)

  layer_array.forEach(function(n)  {

    console.log('icon_scale', size, styles_items.value[n].style)

    if (styles_items.value[n].style != undefined) {

      var layer = map_layers.value[n]

      icon_size.value = size  // new icon size



      map_layers.value[n].setStyle(new Style({
                        image: new Icon({
                        src: layer.getStyle().image_.iconImage_.src_,
                        scale: [icon_size.value, icon_size.value],
                        crossOrigin: undefined,
                      }),
                  }))
    }
  }
  )
}


// Click map event
function mapClick() {


    map.value.on('singleclick', event => { // Bind a click event'



    //console.log('target', map.value)

    var startTime = performance.now()                                            // TIMETIMETIMETIME START

      const coordinate = event.coordinate // Get coordinates
      currentCoordinate.value = coordinate // Save coordinate points
       // Set where the cover appears

      viewResolution.value = map.value.getView().getResolution()

      layers.value = map.value.getLayers().getArray();

      console.log(coordinate)






      //for(var i in layers.value){

          //console.log('i', i)

          //console.log('Properties', this.layers[i].getProperties())
          //console.log('Source', this.layers[i].getSource().serverType_)

          //if (layers.value[i].getSource().serverType_ == "geoserver") {

            //console.log('TILE layer filtered:', layers.value[i])




          if (selected.value !== null) {
                    selected.value.setStyle(undefined);
                    console.log('selected:', selected.value.getProperties())
                    selected.value = null;
          }


              map.value.forEachFeatureAtPixel(event.pixel, function (feature, layer) { // can add layer as argument **

                  console.log('FEATURE', feature.values_)

                  selected.value = feature;
                  //;
                  try {
                  feature.setStyle(new Style({
                        image: new Icon({
                        src: layer.getStyle().image_.iconImage_.src_,
                        scale: [icon_size.value * 1.2, icon_size.value *1.2],
                      }),
                  }))
  
                  }
                  catch {
                    
                    feature.setStyle(
                      new Style({
                          image: new CircleStyle({
                            fill: new Fill({
                              color: 'rgba(255,255,255,0.4)'
                              }),
                            stroke: new Stroke({
                              color: '#3399CC',
                              width: 1.25,
                              }),
                              radius: 7,
                            })
                      })
                    )
                  }


                  return true;
              });

          if (selected.value) {

            keys_list.value = [] // reset list

            console.log(keys_list)

                var keys = selected.value.getKeys() // keys

                for (var n in keys) {

                    var key = keys[n]

                    keys_list.value.push(key)
                }

               var list_of_remove_keys = ['geometry', 'Shape_Leng', 'Shape_Area', 'Area_Km', 'Lat', 'Long', 'coordinates', 'attributes'] //   REMOVE KEYS HERE!

               var keys_list2 = keys_list



                for (var remove_key in list_of_remove_keys) { // loop for remove keys

                  var k3 = list_of_remove_keys[remove_key]

                  var position = keys_list2.value.indexOf(k3) // get index of remove key

                  //console.log('removeKey', k3)

                  //console.log('keylist_length', keys_list2.value.length)

                  //console.log('position', position)

                  //console.log('splice', keys_list.value.splice(position, 1))

                  if (position != -1) {keys_list2.value.splice(position, 1)

                  }
                }


                // Create lists of pupup content strings .replace(/£/g, ".\n\n<hr style='margin-left: 45%; margin-right: 45%; border: 1; '> \n")

                popup_content_info.value = []
                //popup_content_feed.value = '<h3 class="header-orange">' + 'Support given:\n'.bold() + '</h3>' + selected.value.get('attributes')

                for (var k1 in keys_list2.value) { // keys list filter keys and push to popup content



                    var k2 = keys_list2.value[k1]

                    //console.log('key', k2)

                    popup_content_info.value.push(k2.bold() + ':\n ' + selected.value.get(k2))  // push both key and value to final content list


                }



                  overlay.value.setPosition(event.coordinate) // set coordinate of overlay for activation of popup

          }


          else {   // If there is no feature in pixel
            popup_content_info.value = null;
            overlay.value.setPosition(null)
          }

          var endTime = performance.now()                                              // TIMETIMETIMETIME END
          console.log(`Map click took ${endTime - startTime} milliseconds`)

    })
  }








onMounted(() => {
     // Perform map initialization after the element is loaded
  initMap()
  })


  </script>


<style >

.map__x {
  width: 100%;
  height: 100%;
  border: 1px solid #eee;
  }



.mouse-position {
  color: rgb(255, 65, 36);
  font-size: 15px;
  font-family: Roboto, Arial, sans-serif;
  padding-left: 650px
}

.ol-overviewmap {
  bottom: 50px;


}




/*  .zoomslider {
       padding: 0px;

      }
.overviewMap {
    left: 0.5em;
    bottom: 0.5em;
}

*/

.popup {
  background-color: rgb(123, 152, 188, 0.7);
  width: 15vw;
  height: 10vh;
  color: white;
  box-shadow: 0 5px 10px rgb(2 2 2 / 40%);

  padding: 2.8vh;
  overflow:auto;
  transition: all 0.3s ease-in-out;

  border-radius: 0.8vh;
  box-shadow: 0 2.6vh 3vh rgb(2 2 2 / 40%);
  }
  .popup:hover {
  background-color: rgb(23, 9, 46, 0.9);;
  height: 30vh;
}

.popup2 {
  box-sizing: border-box;
  overflow: hidden;
  margin: 8vh auto;
  padding: 1vh;
  padding-right: 3vh;

  background-color: rgb(123, 152, 188);
  border-radius: 0.8vh;
  box-shadow: 0 2.6vh 3vh rgb(2 2 2 / 40%);

  position: relative;
  transition: all 0.3s ease-in-out;
  color: #333;
  font-family: Tahoma, Arial, sans-serif;
  font-size: 25px;
}
.popup2:hover {
  background-color: rgba(0,60,136,0.7);
}

.popup2-text {
    color: white;
    font-size: 14px;
    font-weight: 400;
    font-family: Open Sans,sans-serif;
    letter-spacing: .03125em;
    line-height: 1.5rem;
    white-space: pre-wrap;
}

.popup-close {
  position: absolute;
  top: 0.4vh;
  right: 2vh;
  cursor: pointer;
  color: rgb(255, 255, 255);
  font-size: 15px;
  font-family: Silka, sans-serif;
  border: 2px;
  border-color: rgba(0, 0, 0, 0.849);
  }

.popup-close2 {
  position: absolute;
  top: 0.4vh;
  right: 1vh;
  cursor: pointer;
  color: rgb(255, 255, 255);
  font-size: 15px;
  font-family: Silka, sans-serif;
  border: 2px;
  border-color: rgba(0, 0, 0, 0.849);
  }

.overlay-text-header {
color: white;
font-size: 16px;
font-family: Silka, sans-serif;
font-weight: bold;
}

.overlay-text-feed {
color: white;
font-size: 13px;
font-family: Silka, sans-serif;
white-space: pre-wrap;
line-height: 1.5;
}

.overlay-text-info {
color: rgb(255, 255, 255);
font-size: 13px;
font-family: Silka, sans-serif;
white-space: pre-wrap;
line-height: 1.5;
position:relative;
padding-bottom: 5px;
stroke-width: 3px;
-webkit-text-stroke-color: white;

}

.header {
    grid-area: header;
    font-size: 25px;
    height: 5vh;
    font-family: 'Gill Sans','Gill Sans MT', Calibri, 'Trebuchet MS', sans-serif;
}



.grid-container {
    display: grid;
    grid-template-areas:
    'header header'
    'left right';
}

.left {
    grid-area: left;
    width: 320px;;
    height: 85vh;
    background: rgb(23, 9, 46);
    border-radius: 0.5vh;
    overflow:auto;
}

.right {
    grid-area: right;
    width: 82vw;
    height: 85vh;
    border: 0px solid rgb(0, 0, 0);
    border-radius: 1.5vh;
}

.toc {
    margin-left: 2vh;
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}


/*                    LEGEND                */

.overlay-legend {
display: block;
min-height: 10vh;
max-height: 70vh;
min-width: 17vh;
overflow:auto;
box-sizing: border-box;

margin: 40px auto;
padding: 15px;
border: 1px solid rgba(0,110,172,1);
box-shadow: 0 5px 10px rgb(2 2 2 / 40%);
border-radius: 2.5px;



background-color: rgb(123, 152, 188);
font-family: Tahoma, Arial, sans-serif;

transition: all 0.5s ease-in-out;
position: absolute;
left: 60px;
top: 20px;
}

.overlay-legend:hover {
  background-color: rgba(0,60,136,0.7);
}

.overlay-legend-toggle {
background-color: rgb(123, 152, 188);
height: 25px;
width: 80px;
color: #fff;
border: 0.2px;
border-radius: 1.5px;
border-color: rgba(255, 255, 255, 0.849);
position: absolute;
left: 60px;
top: 9px;

}
.overlay-legend-toggle:hover {
  background-color: rgba(0,60,136,0.7);
  box-shadow: 0 2.6vh 3vh rgb(2 2 2 / 40%);
}


.legend-grid-container {
    display: grid;
    grid-template-areas:
    'legend-header legend-header'
    'legend-left legend-right';
}

.legend-header {
    grid-area: legend-header;
    width: 3vw;
    height: 3vh;
    border: 0px solid rgb(0, 0, 0);

    margin: 0px;
    padding: 0px;

    font-size: 1.17em;
    margin-top: 0.1em;
    margin-bottom: 0.4em;
    margin-left: 0;
    margin-right: 0;
    font-weight: bold;

    color:white;
}

.legend-left {
    grid-area: legend-left;
    width: 3vw;
    height: 5vh;
    border: 0px solid rgb(0, 0, 0);

}

.legend-right {
    grid-area: legend-right;
    width: 10vw;
    height: 6vh;
    border: 0px solid rgb(0, 0, 0);
}

.legend-img {
  padding: 0px;
  margin: 0px;
  height: 40px;
}

.legend-list {
  margin-block-start: 0em;
  margin-block-end: 0em;
  margin-inline-start: 0px;
  margin-inline-end: 0px;
  /*padding-inline-start: 35px */
}

.legend-text {
  color: rgb(255, 255, 255);
  font-size: 13px;
  font-family: Silka, sans-serif;
  font-weight: bold;

  position: relative;
  top: 1em;
}

.legend-text:hover {
  cursor: pointer;
}

/* GENERAL CLASES */

.text-bold-white {
color: rgb(255, 255, 255);
font-size: 13px;
font-weight: bold;
padding-bottom: 5px;
bottom: 5px;
}

.text-bold-black {
color: rgb(0, 0, 0);
font-size: 13px;
font-family: Silka, sans-serif;
font-weight: bold;
}

.text-bold-purple {
color: rgb(23, 9, 46);
font-size: 13px;
font-family: Silka, sans-serif;
font-weight: bold;
}

.text-bold-orange {
color: rgb(255, 65, 36);
font-size: 13px;
font-family: Silka, sans-serif;
font-weight: bold;
}

.header-orange {
color: rgb(255, 65, 36);
font-size: 16px;
font-family: Silka, sans-serif;
font-weight: bold;
font-weight: 700;
font-family: Silka, sans-serif;
margin-top: 3.5vh;
}

.header-orange-attributes {
color: rgb(255, 65, 36);
font-size: 16px;
font-family: Silka, sans-serif;
font-weight: bold;
font-weight: 700;
font-family: Silka, sans-serif;
margin-top: 1vh;
}




::-webkit-scrollbar {

  background-color: rgba(255,255,255,0.4);
  width: 11px;
  border-radius: 10px;
  padding: 10px;
  cursor:pointer;
}

::-webkit-scrollbar-thumb {
  background-color: rgb(123, 152, 188, 0.7);
  border-radius: 10px;
  border: 3px rgba(255,255,255,0.4);
}

::-webkit-scrollbar-track {
    background-color: rgba(255,255,255,0.4);
}

::-webkit-scrollbar-thumb:hover {
  background-color: rgb(123, 152, 188);
}

.hr {
  border: 0;
  border-top: 1px solid #CCC;
}

.hr2 {
  border: 0;
  border-top: 1px dashed #CCC;
}


.slidecontainer {
  width: 70%; /* Width of the outside container */
}

.PDF-export-text {
      font-weight: 400;
    font-family: Open Sans,sans-serif;
}

.home-header {
        color: rgb(0, 0, 0);
        font-size: 3rem;
        letter-spacing: normal;
        font-family: Lato,sans-serif;
        font-weight: 900;
        }


/* NOTES

background-color: rgba(255,255,255,0.4);  Grey background ol default

background-color: rgba(0,60,136,0.5); Blue background ol default

border-radius: 2px 2px 0 0;

box-shadow: 0 5px 10px rgb(173, 173, 173);

rba(255, 65, 36) #orange
rba(23, 9, 46) #purpule

font-size: 15px;

font-weight: 700;


<hr style="margin-left: 0%; margin-right: 25%; height: 3px; background: white;">  DIVIDER

*/


</style>
