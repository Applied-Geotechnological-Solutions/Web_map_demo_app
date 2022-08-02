<template>


<div style="display: flex; justify-content: center; align-items: flex-end;">

<a href="https://agtsolutions.org/">
<img src="https://github.com/Applied-Geotechnological-Solutions/Web_map_demo_data/blob/main/AGT_logo1_2.png?raw=true%22" style="height:120px; object-fit: contain; order: 1;">
</a>
 <div class="home-header" style=" order: 2; margin-left:26%; margin-top:1vh; margin-bottom:2vh; min-width:23%"> Web Map Demo </div> 


  

 

<button style=" order: 3; background-color: rgb(23, 9, 46); border-radius: 1.5vh; margin-top:1vh; margin-bottom:2vh; margin-left:15%; min-width:19% max-width:19%;">
  <a class="button" href="mailto: syver.petersen@agtsolutions.org?subject=Questions about AGT web map"> 
    Questions about our web map product?
  </a>
</button>

    
</div>
    
  
 





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




 const Boat_Clinic_line_cords = [ [ -1810768.222199784126133, 1412706.36043827352114 ], [ -1811105.620444429572672, 1412842.515690139262006 ], [ -1811729.922412696294487, 1413099.566487804753706 ], [ -1812272.148520401213318, 1413429.198431467404589 ], [ -1812716.446872055530548, 1413715.842883033677936 ], [ -1813184.623254484729841, 1414059.808526893844828 ], [ -1813633.686080345185474, 1414336.888884713407606 ], [ -1814350.283038428518921, 1414709.515974414534867 ], [ -1814484.244913649046794, 1414764.771089205984026 ], [ -1814675.335951544810086, 1414817.322861165506765 ], [ -1814904.654102578992024, 1414850.756651097675785 ], [ -1815253.395803335821256, 1414912.867631578585133 ], [ -1815673.793860316975042, 1414984.526414401130751 ], [ -1815936.552386385388672, 1415003.633171428460628 ], [ -1816347.399231055984274, 1415008.412714393576607 ], [ -1816634.035787899512798, 1414989.305954240961 ], [ -1816968.450670192018151, 1414955.860598783707246 ], [ -1817298.089946328662336, 1414888.981409688014537 ], [ -1817680.272022120188922, 1414783.877702547702938 ], [ -1817870.094017820665613, 1414712.9380470267497 ], [ -1818144.785993302473798, 1414607.83497300487943 ], [ -1818345.428243508329615, 1414502.732276933267713 ], [ -1818560.408444128464907, 1414354.637812742730603 ], [ -1818713.283500834368169, 1414251.9200334190391 ], [ -1818875.720901800552383, 1414142.050621625501662 ], [ -1818999.931189627153799, 1414075.161887905560434 ], [ -1819090.689970470964909, 1414051.276350561762229 ], [ -1819152.795114384265617, 1414041.728985075373203 ], [ -1819200.573439833242446, 1414044.11297449329868 ], [ -1819253.127371436683461, 1414051.276350561762229 ], [ -1819382.113265418447554, 1414067.998505984200165 ], [ -1819496.761208986863494, 1414091.884056988405064 ], [ -1819625.758234918117523, 1414125.328423847677186 ], [ -1819838.345066485926509, 1414165.936233126558363 ], [ -1819964.948723364854231, 1414199.369277987396345 ], [ -1820060.494242313085124, 1414235.209179524332285 ], [ -1820060.494242313085124, 1414235.209179524332285 ], [ -1820127.37499238178134, 1414325.253989154007286 ], [ -1820124.981623329920694, 1414473.348304477985948 ], [ -1820036.605079588713124, 1414714.603455843869597 ], [ -1819878.954416726948693, 1415010.79678265703842 ], [ -1819845.547437540255487, 1415159.008642792701721 ], [ -1819845.547437540255487, 1415264.102292194962502 ], [ -1819859.885387954534963, 1415390.710241861408576 ], [ -1819898.101369143696502, 1415491.025102106388658 ], [ -1819957.813144005136564, 1415569.860458617564291 ], [ -1820055.752032005228102, 1415677.35111610032618 ], [ -1820203.851482556434348, 1415803.949449695879593 ], [ -1820332.837376538896933, 1415906.661278040613979 ], [ -1820468.992245727684349, 1416006.977910476503894 ], [ -1820559.762158520752564, 1416097.74685021978803 ], [ -1820633.811883796472102, 1416178.968008247669786 ], [ -1820698.299264813074842, 1416205.239450839813799 ], [ -1820772.34899008879438, 1416221.962839118205011 ], [ -1820824.902921692468226, 1416224.347006440861151 ], [ -1820879.839090398745611, 1416212.403356788447127 ], [ -1820910.897228329908103, 1416181.352172057144344 ], [ -1820937.168628156883642, 1416150.301020351005718 ], [ -1820956.282184726558626, 1416116.854335469892249 ], [ -1820975.384609346510842, 1416095.362693236442283 ], [ -1821004.049378225812688, 1416078.639377473387867 ], [ -1821039.883122312137857, 1416066.695787427248433 ], [ -1821078.099103501532227, 1416066.695787427248433 ], [ -1821109.146109483903274, 1416076.255222050705925 ], [ -1821147.373222621856257, 1416092.967128998134285 ], [ -1821195.140416121575981, 1416102.526572223287076 ], [ -1821225.764408038929105, 1416105.617991881677881 ], [ -1821258.013664521509781, 1416097.256329867988825 ], [ -1821297.420764262787998, 1416074.566920720273629 ], [ -1821329.670020745368674, 1416043.515882577514276 ], [ -1821360.728158676531166, 1416019.628736103419214 ], [ -1821388.19067705492489, 1416000.52131447638385 ], [ -1821415.664327383274212, 1415986.182200907263905 ], [ -1821449.104702417273074, 1415965.877024286892265 ], [ -1821475.376102244714275, 1415947.96741392929107 ], [ -1821512.400964882457629, 1415930.046407174086198 ], [ -1821542.25685231317766, 1415904.97298651561141 ], [ -1821564.954896485898644, 1415889.44755348819308 ], [ -1821574.506108795991167, 1415875.108494461746886 ], [ -1821580.483965451363474, 1415848.837348894914612 ], [ -1821579.281714951153845, 1415822.566226962022483 ], [ -1821575.708359296200797, 1415796.283721330575645 ], [ -1821562.561527434037998, 1415772.396777094574645 ], [ -1821535.099009054945782, 1415774.792315032798797 ], [ -1821484.938446504063904, 1415770.012646674877033 ], [ -1821447.913583865854889, 1415768.814877875847742 ], [ -1821419.248814986553043, 1415770.012646674877033 ], [ -1821400.13525841711089, 1415753.289517235476524 ], [ -1821395.359652262413874, 1415730.600375346606597 ], [ -1821395.359652262413874, 1415703.131593670928851 ], [ -1821404.910864572273567, 1415681.628848635824397 ], [ -1821428.800027296412736, 1415663.71941260015592 ], [ -1821457.464796175714582, 1415652.962348971981555 ], [ -1821489.714052658295259, 1415641.01893302355893 ], [ -1821511.209846330573782, 1415626.691403235774487 ], [ -1821518.378821537597105, 1415606.38647656631656 ], [ -1821521.963309141807258, 1415588.47708666860126 ], [ -1821515.985452485736459, 1415553.833285271655768 ], [ -1821501.65863402094692, 1415518.003173680510372 ], [ -1821486.129565055016428, 1415478.591242413502187 ], [ -1821474.184983692830428, 1415445.156734018586576 ], [ -1821463.520576474955305, 1415413.57022746745497 ], [ -1821458.154977018712088, 1415394.451795167289674 ], [ -1821457.553851768607274, 1415375.949362458661199 ], [ -1821459.346095570363104, 1415343.701315706595778 ], [ -1821462.930583173641935, 1415318.616991255432367 ], [ -1821468.897307880688459, 1415295.928187008248642 ], [ -1821477.257401638897136, 1415266.668895329348743 ], [ -1821488.60085775074549, 1415243.968724312260747 ], [ -1821499.95544581185095, 1415232.630052529042587 ], [ -1821517.265626630047336, 1415216.500400251941755 ], 
 [ -1821552.498245465802029, 1415192.613930142251775 ], [ -1821581.764139595907182, 1415190.823016025824472 ], [ -1821616.395633181324229, 1415195.602589612593874 ], [ -1821659.98834577598609, 1415211.127653840463609 ], [ -1821685.068627051543444, 1415228.443642631638795 ], [ -1821715.525639732833952, 1415250.53922425955534 ], [ -1821741.207046258961782, 1415273.227993255248293 ], [ -1821759.118352327495813, 1415294.73043765174225 ], [ -1821780.614145999774337, 1415328.176181024871767 ], [ -1821799.126577318878844, 1415362.215135152451694 ], [ -1821810.325318092247471, 1415383.147286491468549 ], [ -1821820.778218277962878, 1415407.033912391867489 ], [ -1821827.936061535729095, 1415430.030797041719779 ], [ -1821834.214480816619471, 1415455.41180434403941 ], [ -1821837.498405795311555, 1415476.606388236163184 ], [ -1821840.782330773537979, 1415495.120291917584836 ], [ -1821843.765693126711994, 1415513.337619562866166 ], [ -1821850.63410570868291, 1415533.037898376351222 ], [ -1821853.918030687374994, 1415548.859730065567419 ], [ -1821863.769805622519925, 1415571.856726376805454 ], [ -1821875.714386984705925, 1415578.723888507578522 ], [ -1821907.061955591430888, 1415592.754806185839698 ], [ -1821927.967755962861702, 1415595.743506595259532 ], [ -1821939.311212074477226, 1415593.655979178380221 ], [ -1821968.565974255092442, 1415588.807896954007447 ], [ -1821992.455136979231611, 1415576.567918826360255 ], [ -1822010.065880422713235, 1415564.019950301619247 ], [ -1822021.709899159846827, 1415553.570918807992712 ], [ -1822041.213073946535587, 1415531.475107967620715 ], [ -1822064.501111420569941, 1415501.622405051952228 ], [ -1822073.451198480790481, 1415489.679050984792411 ], [ -1822085.996905092848465, 1415472.36290734517388 ], [ -1822099.733730256557465, 1415458.024053089087829 ], [ -1822121.229523928835988, 1415432.346456570317969 ], [ -1822148.703174256719649, 1415413.843999858479947 ], [ -1822174.373448833590373, 1415398.912001106888056 ], [ -1822198.262611557962373, 1415388.759615176124498 ], [ -1822223.944018083857372, 1415378.014059979701415 ], [ -1822267.403147289762273, 1415359.397566063562408 ], [ -1822311.886415810324252, 1415338.796239198418334 ], [ -1822342.944553741952404, 1415322.369942475110292 ], [ -1822370.529523560777307, 1415304.780126640805975 ], [ -1822377.097373516997322, 1415300.000535191269591 ], [ -1822395.65433263243176, 1415281.988668955396861 ], [ -1822334.061258376808837, 1415340.461683688452467 ], [ -1822288.68743392941542, 1415391.816737759159878 ], [ -1822270.764995911158621, 1415426.448939810041338 ], [ -1822251.662571291206405, 1415467.058557098498568 ], [ -1822237.324620877625421, 1415499.306740032276139 ], [ -1822211.05322105018422, 1415538.718699250835925 ], [ -1822182.388452170882374, 1415559.023578880354762 ], [ -1822158.499289446743205, 1415567.373679543379694 ], [ -1822131.025639118859544, 1415573.35107547394 ], [ -1822108.338726895162836, 1415578.130711647681892 ], [ -1822090.427420826628804, 1415584.096702530048788 ], [ -1822067.72937665367499, 1415587.678579094121233 ], [ -1822052.20030768821016, 1415591.260456097079441 ], [ -1822021.153301705839112, 1415596.04009520355612 ], [ -1821994.881901878397912, 1415599.621973230969161 ], [ -1821965.014882498886436, 1415602.017496997956187 ], [ -1821936.35011361958459, 1415604.40161370462738 ], [ -1821905.303107637213543, 1415599.621973230969161 ], [ -1821883.80731396516785, 1415588.8763404621277 ], [ -1821870.671614051330835, 1415566.187326299259439 ], [ -1821864.693757395725697, 1415535.136828691000119 ], [ -1821859.91815124056302, 1415496.911224689567462 ], [ -1821851.558057481888682, 1415457.499321881216019 ], [ -1821844.38908227509819, 1415433.612654713448137 ], [ -1821830.06226381030865, 1415393.00308392313309 ], [ -1821820.499919550959021, 1415372.698319696821272 ], [ -1821806.173101086169481, 1415340.461683688452467 ], [ -1821791.84628262091428, 1415316.57511222246103 ], [ -1821747.652444775681943, 1415271.186117395525798 ], [ -1821677.187207103706896, 1415219.831275383243337 ], [ -1821630.611132155638188, 1415201.910715374862775 ], [ -1821582.83280670735985, 1415187.583400960313156 ], [ -1821541.032337914453819, 1415193.54931208002381 ], [ -1821493.254012465942651, 1415222.215360896429047 ], [ -1821470.567100242013112, 1415248.497349985875189 ], [ -1821450.262425121385604, 1415285.513472810387611 ], [ -1821445.486818966455758, 1415326.122894176980481 ], [ -1821444.295700415270403, 1415371.511974359396845 ], [ -1821457.431400328408927, 1415425.251185100292787 ], [ -1821478.92719400068745, 1415461.081182919908315 ], [ -1821498.040750569896773, 1415495.724874305306002 ], [ -1821517.143175190314651, 1415538.718699250835925 ], [ -1821526.705519449198619, 1415559.023578880354762 ], [ -1821527.896638000616804, 1415584.096702530048788 ], [ -1821519.536544242175296, 1415619.926895199343562 ], [ -1821511.176450483500957, 1415628.28842054028064 ], [ -1821481.320563053013757, 1415639.034067808417603 ], [ -1821449.071306569967419, 1415643.813714731950313 ], [ -1821402.495231622131541, 1415666.502789199119434 ], [ -1821402.495231622131541, 1415666.502789199119434 ], [ -1821378.995687115937471, 1415682.598468097159639 ], [ -1821374.821206210879609, 1415717.231014925986528 ], [ -1821385.563537072390318, 1415750.665834768442437 ], [ -1821399.901487486436963, 1415762.609295556787401 ], [ -1821430.35850016772747, 1415773.3663995701354 ], [ -1821516.942800106946379, 1415781.123380530625582 ], [ -1821559.934387451503426, 1415790.682721668621525 ], [ -1821570.676718312548473, 1415806.538914941716939 ], [ -1821571.277843562653288, 1415849.533197429962456 ], [ -1821546.197562287095934, 1415888.945629267487675 ], [ -1821505.588212045608088, 1415913.425852492684498 ], [ -1821459.613262348342687, 1415946.267712626606226 ], [ -1821367.051105753751472, 1416002.403537544189021 ], [ -1821340.089525083545595, 1416023.30192545405589 ], [ -1821315.009243807755411, 1416050.771008297102526 ], [ -1821289.327837281627581, 1416072.855804604943842 ], [ -1821267.832043609581888, 1416085.403991768369451 ], [ -1821239.768399980617687, 1416094.358837721869349 ], 
 [ -1821209.912512549664825, 1416097.347589467186481 ], [ -1821192.591199782211334, 1416093.161055620294064 ], [ -1821164.527556153479964, 1416088.392742595635355 ], [ -1821144.824006282957271, 1416080.031085513532162 ], [ -1821120.333718308247626, 1416073.460398953175172 ], [ -1821092.871199929853901, 1416066.296527099097148 ], [ -1821060.621943447273225, 1416059.725843630498275 ], [ -1821014.045868498971686, 1416065.691932897316292 ], [ -1820977.021005861228332, 1416090.183711751364172 ], [ -1820945.361742679495364, 1416122.421175315510482 ], [ -1820920.71560741821304, 1416157.818544497014955 ], [ -1820895.635326142422855, 1416186.782134381355718 ], [ -1820875.03008839674294, 1416199.318963017780334 ], [ -1820855.916531827300787, 1416205.593083099927753 ], [ -1820834.120175529969856, 1416209.768225575331599 ], [ -1820815.318313535070047, 1416212.460394263966009 ], [ -1820780.976250625448301, 1416210.966012460645288 ], [ -1820752.612044371198863, 1416202.604320274898782 ], [ -1820732.006806625286117, 1416194.847227345220745 ], [ -1820671.115045161219314, 1416170.25269338907674 ], [ -1820637.674670126987621, 1416153.825929003767669 ], [ -1820622.746726411627606, 1416143.981282277964056 ], [ -1820606.026538894278929, 1416131.136475894832984 ], [ -1820595.273076083976775, 1416118.59967653779313 ], [ -1820563.023819600930437, 1416082.175684919115156 ], [ -1820503.312044739257544, 1416022.457776308292523 ], [ -1820460.309325446141884, 1415987.813460091827437 ], [ -1820424.486713309073821, 1415950.796447192318738 ], [ -1820373.12390025658533, 1415908.988382734823972 ], [ -1820308.636519240448251, 1415849.282235662918538 ], [ -1820256.082587637007236, 1415814.638124897610396 ], [ -1820193.977443723240867, 1415760.898197723086923 ], [ -1820139.041275016497821, 1415715.508512449683622 ], [ -1820110.376506137195975, 1415688.039744968526065 ], [ -1819966.496064287377521, 1415557.323900212766603 ], [ -1819905.582038925262168, 1415467.742989334277809 ], [ -1819879.310639098053798, 1415368.614554215455428 ], [ -1819864.983820632798597, 1415259.927284910110757 ], [ -1819863.781570132123306, 1415161.997298824368045 ], [ -1819868.557176287285984, 1415103.47901303274557 ], [ -1819890.064101908355951, 1415012.701756001450121 ], [ -1819995.127437319373712, 1414834.51314005558379 ], [ -1820057.232581232441589, 1414693.580528224119917 ], [ -1820097.841931473929435, 1414586.093490002444014 ], [ -1820136.057912663323805, 1414495.329246929846704 ], [ -1820155.171469232533127, 1414440.382625195663422 ], [ -1820162.329312490765005, 1414385.447513529565185 ], [ -1820159.947075387462974, 1414337.675951345590875 ], [ -1820126.506700353231281, 1414261.239332880591974 ], [ -1820028.567812353139743, 1414187.186915348982438 ], [ -1820028.567812353139743, 1414187.186915348982438 ], [ -1819921.344878821400926, 1414102.640545975649729 ], [ -1819782.807772529078647, 1413935.430630588205531 ], [ -1819701.589072046335787, 1413773.001012331340462 ], [ -1819596.492340788478032, 1413472.029526952188462 ], [ -1819543.938409185269848, 1413276.159048208035529 ], [ -1819515.273640305968001, 1413065.963606126839295 ], [ -1819539.16280303010717, 1412879.642829033313319 ], [ -1819582.154390373965725, 1412626.449223293690011 ], [ -1819620.370371563360095, 1412449.68949328828603 ], [ -1819749.35626554582268, 1411928.951974333962426 ], [ -1820247.243820067727938, 1409921.926498941378668 ], [ -1820591.209914669860154, 1409425.081430916441604 ], [ -1821126.267047167988494, 1409195.776425015181303 ], [ -1822196.38131216308102, 1409081.113189495168626 ], [ -1823037.188558074180037, 1409233.993801522767171 ], [ -1823610.47280371049419, 1409463.299106176709756 ], [ -1824260.189011725364253, 1409807.271829567616805 ], [ -1824833.462125412421301, 1410113.018610872561112 ], [ -1825750.701333701843396, 1410380.549660127144307 ], [ -1826744.383636319078505, 1410686.291015296010301 ], [ -1827661.633976557292044, 1410992.046965652378276 ], [ -1828731.748241553315893, 1411488.884379172697663 ], [ -1829343.248468378791586, 1411871.06747549213469 ], [ -1829954.737563255243003, 1412329.686932016164064 ], [ -1830757.32882797694765, 1412788.313573792343959 ], [ -1831407.045035991352051, 1413323.369964324636385 ], [ -1831980.318149678874761, 1413667.342398749664426 ], [ -1832859.352508727926761, 1413858.436141482787207 ], [ -1833547.284697932191193, 1414049.519726331811398 ], [ -1834540.967000549193472, 1414431.713456196477637 ], [ -1835496.422190028009936, 1414737.462912479182705 ], [ -1836451.888511455617845, 1415043.204160491470248 ], [ -1837292.6957573662512, 1414966.765696833841503 ], [ -1837980.627946570515633, 1414622.800788121297956 ], [ -1839012.526230377145112, 1414164.179602007847279 ], [ -1839662.242438391549513, 1413743.777464695973322 ], [ -1840001.321607348276302, 1413494.260658227140084 ], [ -1840249.731051053386182, 1413260.178757789777592 ], [ -1840431.270876639056951, 1413002.20264345407486 ], [ -1840531.603133691241965, 1412801.556127041112632 ], [ -1840608.035096069797873, 1412600.89958026772365 ], [ -1840622.361914535053074, 1412424.140004558488727 ], [ -1840622.361914535053074, 1412233.055629172129557 ], [ -1840603.259489914635196, 1412094.507975610205904 ], [ -1840543.64790259487927, 1411220.054384251125157 ], [ -1840514.983133715577424, 1410947.748196280328557 ], [ -1840471.991546371020377, 1410656.329202379332855 ], [ -1840390.772845888510346, 1410212.038003543624654 ], [ -1840228.3465768720489, 1409648.320802773581818 ], [ -1840046.80675128637813, 1409227.915057017467916 ], [ -1839936.934413873124868, 1409036.817307234043255 ], [ -1839831.826550666242838, 1408883.949127293424681 ], [ -1839693.289444374153391, 1408740.627378465840593 ], [ -1839564.303550391923636, 1408659.415452877758071 ], [ -1839311.09623663360253, 1408563.868273610714823 ], [ -1838967.130142031470314, 1408482.645432665012777 ], [ -1838704.382747912080958, 1408473.099881873466074 ], [ -1838513.291710016317666, 1408496.98087140894495 ], [ -1838288.76029708632268, 1408554.311291889520362 ], [ -1838169.325615414185449, 1408592.527832917869091 ], [ -1838021.226164862746373, 1408664.193960462231189 ], 
 [ -1837839.686339277075604, 1408793.17974555073306 ], [ -1837677.260070260614157, 1408926.944647983415052 ], [ -1837529.160619709407911, 1409032.038738967385143 ], [ -1837447.953051175689325, 1409079.813051936682314 ], [ -1837256.862013279926032, 1409146.701782685006037 ], [ -1837037.095074555836618, 1409204.022067838581279 ], [ -1836807.788055470911786, 1409184.919095261953771 ], [ -1836573.705430230591446, 1409170.583320430479944 ], [ -1836358.725229610921815, 1409132.36602652980946 ], [ -1836110.30465395655483, 1409041.595876277191564 ], [ -1835966.9808095600456, 1408969.940231676911935 ], [ -1835766.338559354422614, 1408860.05641822796315 ], [ -1835589.574339923681691, 1408745.405899272765964 ], [ -1835460.588445941684768, 1408602.084827083861455 ], [ -1835408.034514338010922, 1408473.099881873466074 ], [ -1835336.422685910947621, 1408093.652496506692842 ], [ -1835527.513723806478083, 1407654.144236509455368 ], [ -1835718.604761702241376, 1407233.732840549899265 ], [ -1835814.150280650006607, 1406832.440294639905915 ], [ -1835795.04785603005439, 1406278.275804757140577 ], [ -1835699.502337082056329, 1405743.211080321576446 ], [ -1835508.400167237268761, 1405246.379683942999691 ], [ -1835183.547629204345867, 1404653.98800178267993 ], [ -1834839.581534602446482, 1404252.69645042414777 ], [ -1834170.751770018134266, 1403774.966858049621806 ], [ -1833654.808194089215249, 1403564.759626988554373 ], [ -1832947.762448315741494, 1403392.775081673171371 ], [ -1832068.7392212159466, 1403297.233613758347929 ], [ -1831984.526026431005448, 1403301.144661914790049 ], [ -1831909.875175904715434, 1403328.943905482534319 ], [ -1831827.732523648766801, 1403389.297316634561867 ], [ -1831853.747888646787032, 1403314.610986224375665 ], [ -1831868.074707112042233, 1403279.970276194857433 ], [ -1831881.210407025879249, 1403242.946494546951726 ], [ -1831876.434800870716572, 1403152.183124460978433 ], [ -1831862.107982405461371, 1403099.629436629358679 ], [ -1831842.994425836252049, 1403037.52065369929187 ], [ -1831790.440494232811034, 1402956.313075927086174 ], [ -1831733.110956474207342, 1402884.649455851409584 ], [ -1831690.119369129883125, 1402827.318685289239511 ], [ -1831637.565437526442111, 1402755.666781215928495 ], [ -1831589.798244026722386, 1402717.446422417880967 ], [ -1831546.795524733373895, 1402669.671043634880334 ], [ -1831537.244312423281372, 1402598.008120686979964 ], [ -1831546.795524733373895, 1402526.345372004434466 ], [ -1831575.460293612675741, 1402440.361705742543563 ], [ -1831690.119369129883125, 1402344.811923982342705 ], [ -1831819.10526311211288, 1402282.704731940524653 ], [ -1831928.988732474157587, 1402254.03991821128875 ], [ -1832048.423414146294817, 1402206.265290608163923 ], [ -1832129.630982679780573, 1402153.724691704846919 ], [ -1832272.954827076056972, 1402096.395342965610325 ], [ -1832373.275952179217711, 1402058.175839075120166 ], [ -1832459.270258817123249, 1402053.398404573788866 ], [ -1832564.366990074748173, 1402086.840462346794084 ], [ -1832688.577277902048081, 1402158.502142466371879 ], [ -1832779.347190694650635, 1402220.597670760704204 ], [ -1832860.565891177626327, 1402277.927261050324887 ], [ -1832994.327391314553097, 1402340.0344430252444 ], [ -1833128.088891451945528, 1402373.476825986988842 ], [ -1833271.412735848221928, 1402387.809287443058565 ], [ -1833429.063398709753528, 1402368.699340383289382 ], [ -1833500.71975493314676, 1402344.811923982342705 ], [ -1833581.938455415889621, 1402311.369573544012383 ], [ -1833701.373137088259682, 1402234.930057879304513 ], [ -1833854.248193794628605, 1402148.947241717483848 ], [ -1833978.458481622161344, 1402110.727669697720557 ], [ -1834121.771194068947807, 1402101.172784434398636 ], [ -1834241.205875741085038, 1402110.727669697720557 ], [ -1834322.424576223827899, 1402163.27959400205873 ], [ -1834370.191769723081961, 1402225.375132358400151 ], [ -1834393.268300164956599, 1402283.879146144259721 ], [ -1834401.628393923165277, 1402386.600662738550454 ], [ -1834377.750363148050383, 1402498.866146256448701 ], [ -1834365.805781785864383, 1402559.776564184809104 ], [ -1834351.467831371817738, 1402627.847696864511818 ], [ -1834346.692225216887891, 1402689.955616238527 ], [ -1834347.883343768538907, 1402724.595632540294901 ], [ -1834362.221294182585552, 1402760.421525021782145 ], [ -1834380.132600251119584, 1402786.692353554535657 ], [ -1834409.988487681606784, 1402811.777367937844247 ], [ -1834455.373444078024477, 1402844.023045067908242 ], [ -1834500.758400474442169, 1402869.10810825927183 ], [ -1834642.367924712831154, 1402945.993959986139089 ], [ -1834704.439672779291868, 1402966.962863461347297 ], [ -1834778.489398054778576, 1402982.481457968940958 ], [ -1834841.785660519497469, 1402995.628370658727363 ], [ -1834906.284173485590145, 1403000.405957904178649 ], [ -1834975.547160657122731, 1402996.814215631224215 ], [ -1835026.90997370891273, 1402980.098366877064109 ], [ -1835068.710442502051592, 1402952.630120320944116 ], [ -1835104.54418658814393, 1402920.384324328741059 ], [ -1835134.400074019096792, 1402880.966509202960879 ], [ -1835152.311380087630823, 1402847.523550331126899 ], [ -1835163.064842898398638, 1402805.722755692433566 ], [ -1835173.807173759909347, 1402725.701650849077851 ], [ -1835178.582779914606363, 1402625.384818971622735 ], [ -1835180.976148966932669, 1402526.25415461556986 ], [ -1835188.022672734223306, 1402302.76099646743387 ], [ -1835184.438185130711645, 1402235.283521769801155 ], [ -1835178.460328474873677, 1402195.273716599447653 ], [ -1835169.510241415351629, 1402143.32604260626249 ], [ -1835152.790053898468614, 1402092.564268741523847 ], [ -1835118.748553613666445, 1401995.225505272625014 ], [ -1835061.419015855295584, 1401859.074917008168995 ], [ -1835030.372009872924536, 1401758.749632523860782 ], [ -1835016.034059458877891, 1401675.15119045926258 ], [ -1835013.640690406551585, 1401593.935964223230258 ], [ -1835044.698828338179737, 1401479.279555007815361 ], [ -1835109.197341303341091, 1401357.451907754642889 ], [ -1835372.679444062057883, 1401016.120542115997523 ], [ -1835450.313656941289082, 1400943.264473875053227 ], 
 [ -1835487.338519579032436, 1400896.689131716033444 ], [ -1835516.003288458334282, 1400859.656941513065249 ], [ -1835535.10571307875216, 1400813.081731335958466 ], [ -1835553.028151096077636, 1400759.335056298878044 ], [ -1835586.468526130542159, 1400680.5166579275392 ], [ -1835618.706650664331391, 1400623.190172637579963 ], [ -1835660.518251406261697, 1400563.469501507468522 ], [ -1835696.34086354332976, 1400531.226318077649921 ], [ -1835751.277032250072807, 1400491.811696508899331 ], [ -1835795.470870094839483, 1400467.925800174009055 ], [ -1835845.631432646419853, 1400439.262750089401379 ], [ -1835904.15208895644173, 1400420.154065497685224 ], [ -1835967.459483370417729, 1400407.008301002439111 ], [ -1836012.844439766835421, 1400405.822560172528028 ], [ -1836046.284814801299945, 1400411.785468860995024 ], [ -1836080.916308386949822, 1400416.562637495808303 ], [ -1836154.966033662902191, 1400436.868463155813515 ], [ -1836202.733227162156254, 1400473.900122790597379 ], [ -1836232.60024654190056, 1400507.340389841469005 ], [ -1836251.702671161852777, 1400538.386399614391848 ], [ -1836266.040621576365083, 1400581.381129587301984 ], [ -1836263.647252524038777, 1400623.190172637579963 ], [ -1836237.086422020802274, 1400673.459155019372702 ], [ -1836212.00614074524492, 1400695.555187524296343 ], [ -1836167.812302900012583, 1400718.848391406238079 ], [ -1836131.989690763410181, 1400736.760113797150552 ], [ -1836084.211365314433351, 1400764.237697608536109 ], [ -1836049.579871728783473, 1400788.123833785532042 ], [ -1836007.178277685772628, 1400826.934570743935183 ], [ -1835968.361171246273443, 1400881.274248867062852 ], [ -1835945.073133772006258, 1400940.391280503943563 ], [ -1835928.352946254890412, 1401009.074360521044582 ], [ -1835915.584600660949945, 1401098.941805970622227 ], [ -1835908.12619477766566, 1401160.453619234263897 ], [ -1835905.732825725805014, 1401218.670478532323614 ], [ -1835901.068539061583579, 1401334.625669878907502 ], [ -1835899.276295259594917, 1401434.949511504964903 ], [ -1835909.428632820025086, 1401546.698290561791509 ], [ -1835923.755451285280287, 1401596.866230323910713 ], [ -1835952.420220164582133, 1401651.800222255289555 ], [ -1835997.805176560999826, 1401704.351328572025523 ], [ -1836059.910320474300534, 1401752.125142857432365 ], [ -1836122.015464387601241, 1401773.629085479537025 ], [ -1836196.06518966355361, 1401783.183864592807367 ], [ -1836277.283890146296471, 1401776.012079025385901 ], [ -1836336.995665007969365, 1401740.187382964417338 ], [ -1836396.707439869409427, 1401687.636212783399969 ], [ -1836413.427627386292443, 1401620.753040724666789 ], [ -1836408.652021231595427, 1401553.870020358124748 ], [ -1836382.380621404154226, 1401465.494819836458191 ], [ -1836358.491458680015057, 1401372.331158237298951 ], [ -1836351.322483473224565, 1401291.116765888640657 ], [ -1836351.322483473224565, 1401200.348003765568137 ], [ -1836368.042670990107581, 1401119.134084932040423 ], [ -1836394.325202766340226, 1401090.470402255887166 ], [ -1836422.989971645642072, 1401073.744232750032097 ], [ -1836449.261371473083273, 1401061.806747430004179 ], [ -1836501.815303076524287, 1401054.635137272300199 ], [ -1836563.920446990290657, 1401054.635137272300199 ], [ -1836642.74577842047438, 1401059.412409980548546 ], [ -1836704.850922334240749, 1401054.635137272300199 ], [ -1836781.282884713029489, 1401042.697659684112296 ], [ -1836848.163634781958535, 1401018.81131740892306 ], [ -1836924.606729109305888, 1400985.370470712194219 ], [ -1836924.606729109305888, 1400985.370470712194219 ], [ -1836967.776427638716996, 1400968.7013686504215 ], [ -1837044.208390017738566, 1400937.643504566047341 ], [ -1837138.262227789033204, 1400888.673847691621631 ], [ -1837190.215034141670913, 1400837.321354189887643 ], [ -1837211.120834513101727, 1400788.956143412739038 ], [ -1837222.876172740710899, 1400731.88028001319617 ], [ -1837221.083928938722238, 1400690.67536055110395 ], [ -1837213.324960430851206, 1400647.076194640249014 ], [ -1837198.9870100163389, 1400613.042894678656012 ], [ -1837176.901223043212667, 1400582.589680314296857 ], [ -1837076.524438194930553, 1400458.439855298260227 ], [ -1837013.217043780488893, 1400387.979446722427383 ], [ -1836948.729662764118984, 1400318.704943635500968 ], [ -1836900.951337315607816, 1400268.53917720168829 ], [ -1836868.713212781818584, 1400227.939172744052485 ], [ -1836861.577633422100917, 1400212.627250714926049 ], [ -1836860.085952245164663, 1400204.566547371447086 ], [ -1836858.883701744955033, 1400195.308712585363537 ], [ -1836858.883701744955033, 1400187.544446979649365 ], [ -1836859.484826995059848, 1400178.89088463736698 ], [ -1836860.085952245164663, 1400171.126623349729925 ], [ -1836861.577633422100917, 1400163.658796872245148 ], [ -1836862.167626723181456, 1400158.288803732953966 ], [ -1836862.02291138516739, 1400150.991999334190041 ], [ -1836845.213668275391683, 1400148.050475572468713 ], [ -1836820.133386999834329, 1400140.890488722594455 ], [ -1836805.795436585787684, 1400130.139110459014773 ], [ -1836795.053105724276975, 1400117.004876364488155 ], [ -1836777.130667706485838, 1400096.699270746670663 ], [ -1836777.130667706485838, 1400096.699270746670663 ], [ -1836751.21549024945125, 1400075.059735608287156 ], [ -1836728.82914065127261, 1400061.332662529777735 ], [ -1836698.071565344929695, 1400051.7670397979673 ], [ -1836626.415209121536463, 1400046.989930203417316 ], [ -1836558.933333802502602, 1400040.422830922529101 ], [ -1836530.869690173305571, 1400029.078626405913383 ], [ -1836527.875195871340111, 1400013.550184677122161 ], [ -1836527.28520256979391, 1400001.008864719420671 ], [ -1836533.853052526712418, 1399992.058926009340212 ], [ -1836548.179870991967618, 1399981.307601946406066 ], [ -1836582.610989494016394, 1399968.777696872828528 ], [ -1836616.652489778352901, 1399957.125688007101417 ], [ -1836662.326876851031557, 1399929.055994732538238 ], [ -1836701.043795749312267, 1399888.84409639867954 ], [ -1836727.315195576287806, 1399837.481941292528063 ], [ -1836736.866407886147499, 1399799.265363543760031 ], [ -1836739.259776938473806, 1399741.940589668229222 ], 
 [ -1836723.730707972776145, 1399702.527021911926568 ], [ -1836698.650426697451621, 1399651.176591958850622 ], [ -1836663.629314893390983, 1399599.187792921205983 ], [ -1836644.515758324181661, 1399568.131369794486091 ], [ -1836638.537901669042185, 1399538.272086285287514 ], [ -1836645.706876876065508, 1399495.278875428950414 ], [ -1836661.235945841530338, 1399448.705817877082154 ], [ -1836689.900714720832184, 1399405.701336439698935 ], [ -1836777.086139910155907, 1399337.626236812211573 ], [ -1836859.495958944316953, 1399287.462137190857902 ], [ -1836889.351846375269815, 1399264.774311024695635 ], [ -1836929.961196616757661, 1399240.889407331589609 ], [ -1836962.210453099338338, 1399227.755568758351728 ], [ -1837012.371015650685877, 1399212.227548395516351 ], [ -1837076.858396667055786, 1399214.610335130011663 ], [ -1837107.916534598451108, 1399220.584404153982177 ], [ -1837119.861115960404277, 1399234.915334201185033 ], [ -1837125.82784066721797, 1399263.577215152326971 ], [ -1837118.658865460194647, 1399310.161381715210155 ], [ -1837099.5564408400096, 1399369.87951767956838 ], [ -1837085.218490425962955, 1399436.757588756969199 ], [ -1837066.116065805545077, 1399508.412832966307178 ], [ -1837054.171484443359077, 1399558.577307868516073 ], [ -1837043.418021632824093, 1399624.270096820313483 ], [ -1837033.8668093229644, 1399705.479902371065691 ], [ -1837027.888952667359263, 1399774.752966505009681 ], [ -1837031.473440270870924, 1399818.943702591815963 ], [ -1837042.226903081405908, 1399844.026193157071248 ], [ -1837060.138209149939939, 1399864.331624991027638 ], [ -1837085.218490425962955, 1399879.848587086657062 ], [ -1837122.243353063706309, 1399888.21703292359598 ], [ -1837161.661584753775969, 1399897.77120150742121 ], [ -1837179.572890822310001, 1399898.956921927863732 ], [ -1837234.509059528820217, 1399908.522495148703456 ], [ -1837327.672341373516247, 1399909.708216002676636 ], [ -1837411.273278959095478, 1399908.522495148703456 ], [ -1837479.345147579209879, 1399900.154043555492535 ], [ -1837596.163821217836812, 1399885.412349177990109 ], [ -1837636.172046208754182, 1399874.661063950043172 ], [ -1837660.061208933126181, 1399858.539844022598118 ], [ -1837682.759253105847165, 1399830.470244475174695 ], [ -1837703.063928226474673, 1399806.584883257746696 ], [ -1837720.975234295008704, 1399785.082374125020579 ], [ -1837739.487665614113212, 1399768.368345501367003 ], [ -1837755.016734579810873, 1399748.062979659996927 ], [ -1837781.288134406786412, 1399721.783456166042015 ], [ -1837800.401690975995734, 1399706.255176053848118 ], [ -1837833.240940760355443, 1399687.146965917898342 ], [ -1837999.852822630666196, 1399599.370209600310773 ], [ -1838138.256345533998683, 1399493.716938160359859 ], [ -1838174.078957671066746, 1399465.054833878297359 ], [ -1838224.239520222414285, 1399422.061729606240988 ], [ -1838261.264382860157639, 1399387.425590885337442 ], [ -1838288.738033188506961, 1399364.737687933491543 ], [ -1838305.458220705157146, 1399356.3693904413376 ], [ -1838329.347383429761976, 1399352.789492782205343 ], [ -1838359.203270860249177, 1399353.986592270666733 ], [ -1838386.676921188365668, 1399364.737687933491543 ], [ -1838422.4995333252009, 1399375.488787511596456 ], [ -1838465.502252618549392, 1399394.596795997349545 ], [ -1838498.942627653013915, 1399423.258831901475787 ], [ -1838544.327584049431607, 1399455.500805297400802 ], [ -1838568.205614825012162, 1399484.162900308612734 ], [ -1838582.543565239058807, 1399522.390471272869036 ], [ -1838588.510289945406839, 1399573.740677281515673 ], [ -1838584.925802342128009, 1399634.656457080272958 ], [ -1838571.790102428290993, 1399691.981022685300559 ], [ -1838551.485427307663485, 1399748.108584058703855 ], [ -1838528.798515083733946, 1399798.273466154234484 ], [ -1838516.853933722013608, 1399832.910086755407974 ], [ -1838497.740377152804285, 1399866.349627682706341 ], [ -1838471.468977325363085, 1399890.235037247417495 ], [ -1838433.252996135968715, 1399911.737622958840802 ], [ -1838409.363833411829546, 1399920.09467666875571 ], [ -1838379.507945981109515, 1399922.488921686075628 ], [ -1838350.843177101807669, 1399917.711833008332178 ], [ -1838326.954014377435669, 1399911.737622958840802 ], [ -1838326.954014377435669, 1399911.737622958840802 ], [ -1838289.94028368871659, 1399898.660491817863658 ], [ -1838258.292152456007898, 1399899.264753199880943 ], [ -1838226.644021223532036, 1399904.931129393400624 ], [ -1838195.886445917421952, 1399917.175978240789846 ], [ -1838154.386539749568328, 1399938.370751889189705 ], [ -1838120.946164715569466, 1399981.968870970187709 ], [ -1838102.133170771179721, 1400021.086380414199084 ], [ -1838081.828495650552213, 1400061.093236892484128 ], [ -1838068.692795737413689, 1400096.3230300094001 ], [ -1838067.501677185762674, 1400132.749996230239049 ], [ -1838075.260645693633705, 1400171.571272618835792 ], [ -1838091.980833210749552, 1400209.788332313066348 ], [ -1838098.548683167668059, 1400219.935471630189568 ], [ -1838118.252233038190752, 1400236.661157198017463 ], [ -1838136.17467105621472, 1400245.018302889540792 ], [ -1838151.692608072655275, 1400250.992580386111513 ], [ -1838174.390652245143428, 1400255.769722995813936 ], [ -1838189.318595960736275, 1400259.349730135174468 ], [ -1838209.022145831258968, 1400259.953998898854479 ], [ -1838226.944583848584443, 1400259.349730135174468 ], [ -1838249.631496072513983, 1400253.979719587601721 ], [ -1838280.689634003909305, 1400246.215438549872488 ], [ -1838302.775420977501199, 1400240.834029102465138 ], [ -1838331.440189856803045, 1400230.09401572146453 ], [ -1838350.553746425779536, 1400221.132606274215505 ], [ -1838367.273933943128213, 1400212.171199547126889 ], [ -1838387.57860906352289, 1400207.998331694398075 ], [ -1838422.811227899743244, 1400200.234062751289457 ], [ -1838442.514777770265937, 1400200.234062751289457 ], [ -1838455.060484382556751, 1400202.024062897544354 ], [ -1838465.802815243601799, 1400203.221196815837175 ], [ -1838484.315246562706307, 1400203.81406315555796 ], [ -1838505.81104023498483, 1400206.208331184694543 ], [ -1838524.323471554089338, 1400209.184064576867968 ], [ -1838545.229271925054491, 1400212.775467345258221 ], [ -1838563.741703244158998, 1400216.35546927107498 ], [ -1838583.44525311421603, 1400221.72547297202982 ], [ -1838612.700015294831246, 1400233.525803011842072 ], [ -1838632.114134488860145, 1400244.277218932751566 ], [ -1838647.643203454790637, 1400254.732205084757879 ], [ -1838661.66945929499343, 1400264.879359851591289 ], [ -1838670.029553053434938, 1400271.446509064408019 ], [ -1838679.291334687266499, 1400276.816522789886221 ], [ -1838690.334228173829615, 1400286.678652451606467 ], [ -1838697.803766006371006, 1400297.122252309927717 ], [ -1838705.373491380130872, 1400306.573941392824054 ], [ -1838712.53133463836275, 1400320.004694171948358 ], [ -1838720.301435095490888, 1400341.199757044902071 ], [ -1838726.568722426891327, 1400375.540579763939604 ], [ -1838725.377603875705972, 1400397.636389212450013 ], [ -1838715.314321908168495, 1400430.51790304039605 ], [ -1838698.594134391052648, 1400463.958120936295018 ], [ -1838678.289459270425141, 1400497.398376723052934 ], [ -1838659.777027951320633, 1400540.404448387911543 ], [ -1838654.411428495077416, 1400588.769251784076914 ], [ -1838660.979278451530263, 1400635.948384372051805 ], [ -1838678.890584520529956, 1400684.313344357768074 ], [ -1838693.818528235889971, 1400725.518254932481796 ], [ -1838705.161984347971156, 1400739.25703840656206 ], [ -1838716.00450275070034, 1400743.32736801635474 ], [ -1838733.014120944077149, 1400745.7102783289738 ], [ -1838749.144315160112455, 1400744.820962494472042 ], [ -1838772.732915259199217, 1400742.130212191492319 ], [ -1838796.02095273300074, 1400738.550146487774327 ], [ -1838815.724502603523433, 1400729.885021103080362 ], [ -1838832.144127495586872, 1400716.454081788426265 ], [ -1838846.18151528458111, 1400705.406055643223226 ], [ -1838895.273410724243149, 1400663.619699018076062 ], [ -1838911.103042314993218, 1400655.251033342210576 ], [ -1838929.314911009045318, 1400645.400181965203956 ], [ -1838943.352298798039556, 1400639.733663624385372 ], [ -1838980.076598810730502, 1400623.90846323245205 ], [ -1839015.899210947798565, 1400619.724135729018599 ], [ -1839051.13182978425175, 1400620.921286589698866 ], [ -1839118.290878579486161, 1400629.574978535529226 ], [ -1839172.035928734578192, 1400636.735083935316652 ], [ -1839222.797616536263376, 1400648.683798725018278 ], [ -1839278.924903794657439, 1400673.766994368517771 ], [ -1839317.152016932610422, 1400700.640241721412167 ], [ -1839349.802023582393304, 1400743.224754658062011 ], [ -1839358.162117340834811, 1400764.135084178065881 ], [ -1839361.735472995555028, 1400788.61409835726954 ], [ -1839367.11220440082252, 1400819.671805568737909 ], [ -1839370.095566753996536, 1400869.827107345219702 ], [ -1839370.696692004567012, 1400939.102904203580692 ], [ -1839371.297817254671827, 1401001.800148925278336 ], [ -1839372.488935805857182, 1401046.585605531232432 ], [ -1839370.095566753996536, 1401062.707474423805252 ], [ -1839364.729967297753319, 1401075.249245569109917 ], [ -1839354.57762973732315, 1401088.395307773491368 ], [ -1839340.239679323276505, 1401096.75269468408078 ], [ -1839326.502854159334674, 1401101.529974212637171 ], [ -1839306.799304288811982, 1401102.72714462177828 ], [ -1839289.489123470848426, 1401102.122858599992469 ], [ -1839264.998835496138781, 1401096.15981040452607 ], [ -1839241.109672771766782, 1401087.791022044606507 ], [ -1839204.084810133790597, 1401078.236468208255246 ], [ -1839171.245560350129381, 1401071.076255668420345 ], [ -1839146.165279074572027, 1401069.286202805116773 ], [ -1839104.954803582280874, 1401069.879086543573067 ], [ -1839059.580979135353118, 1401078.840753742028028 ], [ -1839013.594897488830611, 1401093.172585949068889 ], [ -1838952.090878825169057, 1401124.219212135532871 ], [ -1838936.561809859704226, 1401143.932639552513137 ], 
 [ -1838927.600590850925073, 1401169.609141981462017 ], [ -1838923.426109945867211, 1401192.891318476293236 ], [ -1838925.808347048936412, 1401212.000503513030708 ], [ -1838932.977322255959734, 1401231.713989592855796 ], [ -1838943.730785066727549, 1401252.613263660110533 ], [ -1838976.236076378496364, 1401289.782767487922683 ], [ -1838998.922988601727411, 1401318.446643840521574 ], [ -1839018.036545171169564, 1401359.059550549136475 ], [ -1839014.452057567890733, 1401399.661111440742388 ], [ -1838990.562894843751565, 1401430.719461967702955 ], [ -1838949.964676551055163, 1401453.40895310905762 ], [ -1838897.410744947614148, 1401468.93815258378163 ], [ -1838817.394294965313748, 1401478.492832975229248 ], [ -1838700.341850396478549, 1401484.467360235517845 ], [ -1838598.829606742132455, 1401473.715492392191663 ], [ -1838514.026418655645102, 1401461.766443438827991 ], [ -1838423.256505863042548, 1401471.321121509186924 ], [ -1838296.663980932440609, 1401514.317211124580353 ], [ -1838242.918930777581409, 1401559.707741290796548 ], [ -1838214.254161898279563, 1401615.838855237700045 ], [ -1838204.70294958841987, 1401673.167269590077922 ], [ -1838205.894068139838055, 1401731.692991331918165 ], [ -1838192.75836822623387, 1401791.404625710332766 ], [ -1838171.251442604931071, 1401829.623783779796213 ], [ -1838115.124155347235501, 1401858.288184843724594 ], [ -1838073.32368655432947, 1401870.225992551306263 ], [ -1838006.442936485633254, 1401886.95261377422139 ], [ -1837958.664611037122086, 1401904.865045522572473 ], [ -1837945.217216548975557, 1401914.009397433837876 ], [ -1837930.289272833615541, 1401930.736043495126069 ], [ -1837920.136935273418203, 1401955.82032940373756 ], [ -1837915.361329118255526, 1401994.632602929836139 ], [ -1837910.040257458807901, 1402035.62270638672635 ], [ -1837909.739694833289832, 1402056.522550198948011 ], [ -1837913.9253076876048, 1402074.435084971599281 ], [ -1837924.36707592359744, 1402092.359032636042684 ], [ -1837942.879507242236286, 1402112.061705237720162 ], [ -1837958.408576208166778, 1402124.603922663722187 ], [ -1837985.581663910765201, 1402136.24538561492227 ], [ -1837997.826807898236439, 1402139.529168355744332 ], [ -1838012.153626363025978, 1402141.022833543131128 ], [ -1838029.764369806973264, 1402141.319286183686927 ], [ -1838057.839145384728909, 1402140.122073610778898 ], [ -1838093.071764220949262, 1402135.356027852511033 ], [ -1838130.987182785291225, 1402129.08491618395783 ], [ -1838163.837564517976716, 1402121.023689144523814 ], [ -1838198.602641493082047, 1402108.002588739618659 ], [ -1838235.6275041308254, 1402089.485727314837277 ], [ -1838265.483391561545432, 1402076.35062084486708 ], [ -1838292.945909940171987, 1402062.611214521341026 ], [ -1838318.037323164986446, 1402052.463441349333152 ], [ -1838346.690960095264018, 1402039.92125461413525 ], [ -1838378.350223276996985, 1402032.156503499252722 ], [ -1838401.638260750565678, 1402029.18058621417731 ], [ -1838423.134054422844201, 1402026.786170224193484 ], [ -1838447.023217147216201, 1402029.18058621417731 ], [ -1838470.901247922098264, 1402032.156503499252722 ], [ -1838490.014804491307586, 1402040.525559852365404 ], [ -1838506.734992008656263, 1402050.080425458028913 ], [ -1838519.280698621179909, 1402062.018310824176297 ], [ -1838532.416398534551263, 1402076.943524828413501 ], [ -1838543.158729396061972, 1402096.657589133828878 ], [ -1838555.704436008818448, 1402133.679930602898821 ], [ -1838561.671160715166479, 1402168.912198953330517 ], [ -1838560.413250469369814, 1402193.540605718502775 ], [ -1838554.735956438817084, 1402212.946892580017447 ], [ -1838541.299693900160491, 1402237.130655723391101 ], [ -1838524.290075706783682, 1402262.215202341089025 ], [ -1838491.139131348580122, 1402304.915991022950038 ], [ -1838473.22782527981326, 1402321.038547882810235 ], [ -1838453.824838034342974, 1402331.482868066756055 ], [ -1838431.126793861854821, 1402338.654788764193654 ], [ -1838398.287544077495113, 1402344.629490327090025 ], [ -1838349.908093379111961, 1402348.506205731537193 ], [ -1838318.560524771455675, 1402349.99988150736317 ], [ -1838253.928428416838869, 1402350.843637322541326 ], [ -1838223.471415736246854, 1402356.818341358099133 ], [ -1838194.205521606607363, 1402377.125502864364535 ], [ -1838166.141877977410331, 1402402.198766496032476 ], [ -1838141.662721951957792, 1402436.245533341541886 ], [ -1838115.981315425829962, 1402473.861213307827711 ], [ -1838080.147571339504793, 1402515.661537206964567 ], [ -1838041.330464900005609, 1402540.153407561359927 ], [ -1838012.665696020703763, 1402563.436666313093156 ], [ -1837953.054108701413497, 1402626.753084441414103 ], [ -1837932.749433580553159, 1402657.81272772536613 ], [ -1837919.613733666948974, 1402699.613312541972846 ], [ -1837898.117939994670451, 1402755.746597052784637 ], [ -1837882.58887102920562, 1402803.522115390980616 ], [ -1837858.710840253857896, 1402850.100469780620188 ], [ -1837840.788402236066759, 1402876.371378271142021 ], [ -1837803.763539598323405, 1402889.506841298658401 ], [ -1837776.301021219231188, 1402891.901327378116548 ], [ -1837744.051764736650512, 1402881.148946176748723 ], [ -1837744.051764736650512, 1402881.148946176748723 ], [ -1837713.795127138728276, 1402854.706998917507008 ], [ -1837698.266058173263445, 1402828.424707452533767 ], [ -1837685.130358259892091, 1402806.931397585198283 ], [ -1837668.410170742776245, 1402787.821178743150085 ], [ -1837648.105495621683076, 1402761.538946938700974 ], [ -1837631.38530810456723, 1402743.625994574977085 ], [ -1837589.58483931212686, 1402719.738274277886376 ], [ -1837563.313439484685659, 1402710.183191582327709 ], [ -1837532.255301553523168, 1402707.800122513435781 ], [ -1837496.432689415989444, 1402710.183191582327709 ], [ -1837471.341276191640645, 1402725.713053100276738 ], [ -1837451.047733019804582, 1402757.958636048482731 ], [ -1837437.900901157176122, 1402818.869589512934908 ], [ -1837431.93417645059526, 1402880.977911513997242 ], [ -1837439.103151657618582, 1402938.308786484180018 ], [ -1837451.047733019804582, 1402984.887353953905404 ], [ -1837478.510251398663968, 1403019.516314630862325 ], [ -1837521.512970692012459, 1403073.267109982203692 ], [ -1837564.504558036569506, 1403125.809348377166316 ], [ -1837603.9116577769164, 1403156.869517751736566 ], [ -1837673.185776897706091, 1403187.918317445553839 ], [ -1837792.620458569610491, 1403230.916911204578355 ], [ -1837865.479065293679014, 1403251.213272678898647 ], [ -1837933.550933914259076, 1403276.298682587686926 ], [ -1837977.744771759025753, 1403312.125245965085924 ], [ -1838040.094818552490324, 1403359.34294427302666 ], [ -1838052.039399914676324, 1403371.292762561002746 ], [ -1838069.950705982977524, 1403383.231183189200237 ], [ -1838084.87864969833754, 1403388.008833297993988 ], [ -1838100.997711965348572, 1403389.206096572801471 ], [ -1838117.127906181151047, 1403386.81157007208094 ], [ -1838134.44921894883737, 1403380.848059677984565 ], [ -1838151.169406465720385, 1403370.688430030597374 ], [ -1838166.687343482393771, 1403355.169631013646722 ], [ -1838191.177631456637755, 1403332.467273337766528 ], [ -1838213.274550379486755, 1403313.368116069817916 ], [ -1838249.097162517020479, 1403287.085415416397154 ], [ -1838279.554175197845325, 1403265.591770927654579 ], [ -1838316.579037835588679, 1403245.283997094491497 ], [ -1838343.451562913367525, 1403238.123258171137422 ], [ -1838369.132969439029694, 1403233.938559639267623 ], [ -1838387.044275508029386, 1403234.542889362899587 ], [ -1838410.33231298183091, 1403233.642096004914492 ], [ -1838420.64049782929942, 1403232.741302674869075 ], [ -1838430.937550727743655, 1403231.851911819074303 ], [ -1838439.186324995243922, 1403230.802886741003022 ], [ -1838447.423967314185575, 1403226.184896353865042 ], [ -1838460.938153496012092, 1403212.741417348617688 ], [ -1838476.155527888098732, 1403197.519164812518284 ], [ -1838493.7774032803718, 1403183.186309503391385 ], [ -1838514.082078400999308, 1403172.137358975596726 ], [ -1838525.726097137900069, 1403160.495486796367913 ], [ -1838535.878434698330238, 1403138.397606909042224 ], [ -1838550.505815788870677, 1403129.743178590433672 ], [ -1838563.94207832752727, 1403123.175392789766192 ], [ -1838570.209365658927709, 1403120.78088791272603 ], [ -1838560.357590724015608, 1403121.681677820393816 ], [ -1838551.106941038975492, 1403125.558495453326032 ], [ -1838542.145722030196339, 1403130.940431367838755 ], [ -1838537.069553249981254, 1403134.224324951414019 ], [ -1838532.293947094818577, 1403144.668933364795521 ], [ -1838528.408896866254508, 1403154.22415697039105 ], [ -1838522.442172159673646, 1403165.56956372410059 ], [ -1838506.022547267610207, 1403180.198873554822057 ], [ -1838492.586284728720784, 1403186.4702089112252 ], [ -1838479.740015491377562, 1403196.926238282117993 ], [ -1838472.582172233145684, 1403204.679893865482882 ], [ -1838463.921515849651769, 1403214.839466255391017 ], [ -1838455.56142209097743, 1403224.987639704719186 ], [ -1838452.578059737803414, 1403230.666057385504246 ], [ -1838451.676371862646192, 1403241.110699994023889 ], [ -1838453.468615663936362, 1403270.369401954114437 ], [ -1838457.654228517785668, 1403316.948494459269568 ], [ -1838461.528146797791123, 1403354.280236460734159 ], [ -1838477.658341013360769, 1403456.389748055953532 ], [ -1838493.176278030266985, 1403513.721742915920913 ], [ -1838523.043297409778461, 1403573.436988282948732 ], [ -1838552.899184840498492, 1403610.461185375461355 ], [ -1838594.699653633637354, 1403637.930045387707651 ], [ -1838642.466847132891417, 1403661.818510271143168 ], [ -1838691.43629113282077, 1403684.509719204856083 ], [ -1838741.59685368440114, 1403715.559075137134641 ], [ -1838777.430597770726308, 1403745.422588648274541 ], [ -1838793.059854277875274, 1403764.864105701446533 ], [ -1838813.954522700048983, 1403804.271701057441533 ], [ -1838835.461448321584612, 1403860.407090723747388 ], [ -1838846.20377918262966, 1403899.814813949400559 ], 
 [ -1838856.957241993397474, 1403923.110550140263513 ], [ -1838877.261917114024982, 1403991.789178745821118 ], [ -1838908.308923096396029, 1404089.716080839047208 ], [ -1838948.918273337883875, 1404192.432492121821269 ], [ -1839018.192392458207905, 1404254.532308043446392 ], [ -1839082.679773475043476, 1404299.915657033445314 ], [ -1839149.560523543506861, 1404366.804911529179662 ], [ -1839186.106712370645255, 1404471.996724775526673 ], [ -1839253.276893115835264, 1404620.80529783340171 ], [ -1839310.595298925181851, 1404711.573195381555706 ], [ -1839391.813999407691881, 1404759.351887091761455 ], [ -1839473.032699890434742, 1404902.6656215172261 ], [ -1839492.13512451085262, 1405065.091738587478176 ], [ -1839473.032699890434742, 1405208.418365519726649 ], [ -1839430.029980597086251, 1405299.188075906829908 ], [ -1839320.157643184065819, 1405370.846170746255666 ], [ -1839238.938942701322958, 1405409.05857117427513 ], [ -1839105.177442564396188, 1405394.736042249482125 ], [ -1839052.623510960955173, 1405351.734287515515462 ], [ -1838990.518367047188804, 1405308.743998899823055 ], [ -1838942.75117354770191, 1405284.854197239037603 ], [ -1838856.756866909796372, 1405265.742369880899787 ], [ -1838751.660135652171448, 1405275.298282009083778 ], [ -1838679.992647479055449, 1405346.95631864760071 ], [ -1838660.890222859103233, 1405447.282424550270662 ], [ -1838699.106204048497602, 1405585.821502757491544 ], [ -1838770.762560272123665, 1405729.139263773802668 ], [ -1838894.972848099190742, 1405944.117214997299016 ], [ -1838942.75117354770191, 1406054.001523394603282 ], [ -1838957.077992012957111, 1406163.87483891681768 ], [ -1838940.090637717861682, 1406270.874851612374187 ], [ -1838861.265306286979467, 1406390.305252228863537 ], [ -1838686.894455908564851, 1406442.853413789300248 ], [ -1838557.897429977310821, 1406447.631560744484887 ], [ -1838395.471160960849375, 1406435.691896657925099 ], [ -1838192.435541702900082, 1406423.740833739517257 ], [ -1838089.721047547878698, 1406469.127529801568016 ], [ -1838070.618622928159311, 1406545.566638899967074 ], [ -1838070.618622928159311, 1406545.566638899967074 ], [ -1838085.780337574193254, 1406656.981301384977996 ], [ -1838122.805200211936608, 1406725.061912065837532 ], [ -1838244.622118986910209, 1406869.571228516055271 ], [ -1838335.392031779512763, 1406984.225827234331518 ], [ -1838418.992969365557656, 1407055.888029620051384 ], [ -1838476.322507124161348, 1407089.335772393038496 ], [ -1838528.876438727602363, 1407103.659127005841583 ], [ -1838581.419238381786272, 1407113.215636594453827 ], [ -1838641.142145192250609, 1407106.053956084884703 ], [ -1838708.022895261179656, 1407075.001020827330649 ], [ -1838765.352433019783348, 1407032.008212016196921 ], [ -1838815.512995570898056, 1406977.064178169704974 ], [ -1838856.122345812385902, 1406931.676697607152164 ], [ -1838915.834120674058795, 1406888.684098521247506 ], [ -1838985.108239794848487, 1406838.518543196609244 ], [ -1839044.820014655822888, 1406802.687685234239325 ], [ -1839087.82273394963704, 1406783.586274989647791 ], [ -1839142.758902655914426, 1406764.473473344929516 ], [ -1839214.415258879773319, 1406742.977288258960471 ], [ -1839262.193584328284487, 1406740.582488772459328 ], [ -1839333.805412755347788, 1406750.61784030799754 ], [ -1839388.741581461858004, 1406788.832034177612513 ], [ -1839443.677750168601051, 1406855.715545922052115 ], [ -1839474.735888099996373, 1406913.042762321420014 ], [ -1839491.456075617112219, 1407006.20115301804617 ], [ -1839489.062706565018743, 1407132.796227406244725 ], [ -1839503.400656979298219, 1407378.826265803305432 ], [ -1839486.680469461949542, 1407689.360307598486543 ], [ -1839455.622331530787051, 1407827.898551526712254 ], [ -1839370.551976666320115, 1408002.417982565006241 ], [ -1839277.39982677064836, 1408113.496040827594697 ], [ -1839173.49421406397596, 1408150.514550736872479 ], [ -1839061.228507598862052, 1408195.903914947761223 ], [ -1839020.619157357374206, 1408224.563114979071543 ], [ -1838999.123363685794175, 1408273.533575641922653 ], [ -1839001.527864686911926, 1408344.776960402727127 ], [ -1839023.023658358491957, 1408394.933684288989753 ], [ -1839075.577589962165803, 1408435.544955302262679 ], [ -1839163.954133703140542, 1408471.366401058156043 ], [ -1839254.724046496208757, 1408502.42081300332211 ], [ -1839390.878915685229003, 1408559.751244110055268 ], [ -1839510.313597357599065, 1408605.141241147648543 ], [ -1839608.252485357224941, 1408655.298409973736852 ], [ -1839729.746577609330416, 1408764.599822436692193 ], [ -1839849.181259281001985, 1408941.360136398579925 ], [ -1839930.399959764210507, 1409108.564352978719398 ], [ -1840059.3858537459746, 1409414.315255052875727 ], [ -1840140.604554228717461, 1409677.061256118351594 ], [ -1840212.260910452576354, 1409868.163177286274731 ], [ -1840276.837347061606124, 1410120.774091431405395 ], [ -1840324.615672509884462, 1410388.293806240428239 ], [ -1840348.493703284999356, 1410617.608138179406524 ], [ -1840424.936797613045201, 1410889.911258080974221 ], [ -1840453.601566492579877, 1411076.230779665987939 ], [ -1840482.266335371881723, 1411286.434747591614723 ], [ -1840544.371479285182431, 1411625.627065749140456 ], [ -1840568.249510060064495, 1411878.82342140795663 ], [ -1840603.259489914635196, 1412094.507975610205904 ], [ -1840592.138672784436494, 1412523.760324611561373 ], [ -1840249.731051053386182, 1413260.178757789777592 ], [ -1839827.908104590373114, 1413910.31329497275874 ], [ -1839636.817066694842651, 1414521.804498468292877 ], [ -1839655.930623264051974, 1415228.854298616060987 ], [ -1839923.453623538371176, 1416069.661721461685374 ], [ -1840401.192350226687267, 1416967.794631059281528 ], [ -1841089.1245394309517, 1417560.184373264433816 ], [ -1842216.56834218534641, 1418248.11901844991371 ], [ -1843592.432720593642443, 1418764.063541901065037 ], [ -1844471.455947693670169, 1418916.937747237505391 ], [ -1845541.581344638951123, 1418974.265780991641805 ], [ -1846955.661704236408696, 1418744.954322618199512 ], [ -1848064.003082370618358, 1418286.336818143259734 ], [ -1849057.685384987853467, 1417789.486513127340004 ], [ -1850089.583668794250116, 1417025.118833014508709 ], [ -1851732.982179426122457, 1415763.909729977836832 ], [ -1852803.096444421913475, 1415190.629095395328477 ], [ -1854140.744841641513631, 1414349.824156390037388 ], [ -1855554.825201238738373, 1413661.890091574750841 ], [ -1857274.666806198656559, 1412973.960801706649363 ], [ -1858688.747165796812624, 1412400.677837451454252 ], [ -1859731.988905714824796, 1411929.476642236113548 ], [ -1860687.44409519364126, 1411451.747598298592493 ], [ -1861566.467322293436155, 1411145.998256060294807 ], [ -1862254.410643446957693, 1410782.917215240653604 ], [ -1862999.661238460335881, 1410572.71685944753699 ], [ -1863458.286408580141142, 1410534.497738071484491 ], [ -1863526.781291265040636, 1410542.515651675174013 ], [ -1863573.357366212643683, 1410561.619508715346456 ], [ -1863603.213253643596545, 1410584.316047593252733 ], [ -1863630.686903971247375, 1410612.977581348270178 ], [ -1863667.711766608990729, 1410657.161791362101212 ], [ -1863696.376535488292575, 1410696.578633799217641 ], [ -1863715.478960108710453, 1410718.077715169638395 ], [ -1863729.816910522757098, 1410740.774375216569752 ], [ -1863729.816910522757098, 1410716.880153192672879 ], [ -1863725.041304367594421, 1410691.799795043421909 ], [ -1863716.681210609152913, 1410671.498293023556471 ], [ -1863697.567654039943591, 1410638.057872044388205 ], [ -1863682.03858507424593, 1410616.558849374996498 ], [ -1863662.936160453828052, 1410593.862285527633503 ], [ -1863640.238116281339899, 1410571.177144552348182 ], [ -1863625.911297816084698, 1410561.619508715346456 ], [ -1863604.415504144271836, 1410547.29446608806029 ], [ -1863572.166247661691159, 1410534.15557964425534 ], [ -1863544.692597333341837, 1410524.597955879988149 ], [ -1863489.756428627297282, 1410512.656633193371817 ], [ -1863457.507172144250944, 1410509.075377861969173 ], [ -1863439.595866075716913, 1410511.459079611115158 ], [ -1863357.186047041555867, 1410511.459079611115158 ], [ -1863206.704359387047589, 1410515.040335232159123 ], [ -1863025.16453380137682, 1410522.202847787644714 ], [ -1862986.94855261198245, 1410525.795509999152273 ], [ -1862939.170227163471282, 1410537.736838039243594 ], [ -1862887.818546060705557, 1410556.840691965771839 ], [ -1862819.735545491566882, 1410581.920934620313346 ], [ -1862744.494701664429158, 1410610.593871344579384 ], [ -1862615.508807682199404, 1410649.999245824525133 ], [ -1862239.29345659702085, 1410757.494639540789649 ], [ -1861950.26353070163168, 1410886.478227080777287 ], [ -1861830.82884902972728, 1410946.197056237841025 ], [ -1861718.563142564380541, 1410996.358229480218142 ], [ -1861568.081454910337925, 1411063.239927389891818 ], [ -1861372.938387549482286, 1411150.834196496522054 ], [ -1861269.032774843042716, 1411197.414380638394505 ], [ -1861136.462393257301301, 1411240.413294130237773 ], [ -1861031.365661999443546, 1411270.273054210236296 ], [ -1860790.114061552332714, 1411353.875982118071988 ], [ -1860675.454986035358161, 1411408.805402701254934 ], [ -1860188.165047036251053, 1411645.290508998790756 ], [ -1859709.190674000419676, 1411880.808031427208334 ], [ -1859183.684753812616691, 1412114.901711075101048 ], [ -1858667.741177883930504, 1412344.206748122815043 ], [ -1857960.69543211045675, 1412640.398881556699052 ], [ -1856388.964409651234746, 1413266.224120019003749 ], [ -1855495.603232136694714, 1413581.53141439659521 ], [ -1855304.512194241164252, 1413667.524902772624046 ], [ -1854759.892717484151945, 1413915.948154038051143 ], [ -1854005.079778211191297, 1414269.463588606566191 ], [ -1853154.721319990698248, 1414737.645423183450475 ], [ -1852122.823036184534431, 1415368.249524926068261 ], [ -1850422.094987793825567, 1416591.237314485246316 ], [ -1849432.720749470405281, 1417319.008381046354771 ], [ -1848439.038446853403002, 1417949.611205815337598 ], 
 [ -1847464.469700805377215, 1418446.452831053873524 ], [ -1846241.469247154192999, 1418809.526421425398439 ], [ -1844980.263944262871519, 1418885.963543092133477 ], [ -1844043.911179404472932, 1418771.307935769902542 ], [ -1843413.297396009555086, 1418656.652779504656792 ], [ -1839075.499666318763047, 1416764.839341755723581 ], [ -1833820.451596389524639, 1414586.378661500522867 ], [ -1833492.994182272115722, 1414457.139223617734388 ], [ -1833187.244068859377876, 1414285.159261987078935 ], [ -1828811.230357979657128, 1411858.281575020169839 ], [ -1825333.342298819450662, 1410214.889292553532869 ], [ -1824951.160223028156906, 1410004.681605028919876 ], [ -1824260.189011725364253, 1409807.271829567616805 ], [ -1823288.648155826842412, 1409488.731964519713074 ], [ -1822084.772390693891793, 1409297.643920477246866 ], [ -1821186.635607024887577, 1409374.078988711116835 ], [ -1820708.896880336804315, 1409660.717971057863906 ], [ -1820422.26032349281013, 1409909.14144894387573 ], [ -1820212.055729028303176, 1410291.32675168174319 ], [ -1819925.419172184308991, 1410864.602643649326637 ], [ -1819619.669058771571144, 1411590.748438446782529 ], [ -1819390.362039686646312, 1412144.922045504907146 ], [ -1819103.725482842884958, 1412679.978078971151263 ], [ -1818683.316293913172558, 1413406.134682460688055 ], [ -1818186.475142604671419, 1414189.616543447598815 ], [ -1817746.963529054541141, 1414610.013684941921383 ], [ -1816772.394783006515354, 1414877.540212900843471 ], [ -1815396.53040459821932, 1414801.113589538028464 ], [ -1814269.086601844057441, 1414590.907185264397413 ], [ -1813695.802356207277626, 1414266.052974647376686 ], 
 [ -1812969.654185813851655, 1413816.984673816245049 ], [ -1812587.472110022557899, 1413549.456444079289213 ], 
 [ -1812281.721996609820053, 1413367.911862501641735 ], [ -1811909.09113312792033, 1413148.157281087944284 ], 
 [ -1811488.68194419844076, 1412928.404350293800235 ], [ -1811030.067906028125435, 1412775.527272682404146 ], 
 [ -1810768.222199784126133, 1412706.36043827352114 ] ]

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
      Type: 'Mobile Boat Clinic',
      geometry: Boat_Clinic_position,
    })
  

  const Water_T_point_features = new Feature({
      type: 'Emergency Water Trucking',
      geometry: Water_T_position,
    })
  

// LAYERS Movement

  const Boat_Clinic_Point = new VectorLayer({
      source: new VectorSource({
          features: [Boat_Clinic_point_features],
          crossOrigin: undefined,
      }),
      //style: Boat_Clinic_style,
      zIndex:2,
  })

  const Boat_Clinic_Stops = new VectorLayer({
      source: new VectorSource({
        format: new GeoJSON(),
        url: 'https://raw.githubusercontent.com/Applied-Geotechnological-Solutions/Web_map_demo_data/main/river_stops_new.json.geojson',
        crossOrigin: undefined,
      }),
  })

  const Boat_Clinic_icon_point = new VectorLayer({
      source: new VectorSource({
        format: new GeoJSON(),
        url: 'https://raw.githubusercontent.com/Applied-Geotechnological-Solutions/Web_map_demo_data/main/Icon_random_point.json.geojson',
        crossOrigin: undefined,
      }),
      style: new Style({
        image: new Icon({
        src: 'https://github.com//Applied-Geotechnological-Solutions/Web_map_demo_data/blob/main/icons/Movement/1.boatc.png?raw=true',
        crossOrigin: undefined,
        scale: [0.1, 0.1],
            }),
          }),
      }) 

  const Water_T_Point = new VectorLayer({
      source: new VectorSource({
          features: [Water_T_point_features]
      }),
      //style: Water_T_style,
      zIndex:2,
  })
  const Water_T_Stops = new VectorLayer({
      source: new VectorSource({
        format: new GeoJSON(),
        url: 'https://raw.githubusercontent.com/Applied-Geotechnological-Solutions/Web_map_demo_data/main/water_truck_stops.json.geojson',
        crossOrigin: undefined,
      }),
  })

  const Water_T_icon_point = new VectorLayer({
      source: new VectorSource({
        format: new GeoJSON(),
        url: 'https://raw.githubusercontent.com/Applied-Geotechnological-Solutions/Web_map_demo_data/main/Icon_random_point.json.geojson',
        crossOrigin: undefined,
      }),
      style: new Style({
        image: new Icon({
        src: 'https://github.com//Applied-Geotechnological-Solutions/Web_map_demo_data/blob/main/icons/Movement/2.wtruck.png?raw=true',
        crossOrigin: undefined,
        scale: [0.1, 0.1],
            }),
          }),
      })
  

  // ADD EXTRA LAYERS
  map.value.addLayer(Boat_Clinic_Lines);
  map.value.addLayer(Boat_Clinic_Point);
  map.value.addLayer(Boat_Clinic_Stops);
  map.value.addLayer(Boat_Clinic_icon_point);
  

  map.value.addLayer(Water_T_Lines);
  map.value.addLayer(Water_T_Point);
  map.value.addLayer(Water_T_Stops);
  map.value.addLayer(Water_T_icon_point);
  
  

    var distance = 0;
    var distance2 = 0;
    var distance3 = 0.0001;
    var lastTime;

    function moveFeature(event) {

      // style called again to enable scaling of icon

      var Boat_Clinic_style = new Style({
      image: new Icon({
      src: 'https://github.com//Applied-Geotechnological-Solutions/Web_map_demo_data/blob/main/icons/Movement/1.boatc.png?raw=true',
      crossOrigin: undefined,
      scale: [icon_size.value, icon_size.value],
            }),
          })

      var Water_T_style = new Style({
      image: new Icon({
      src: 'https://github.com//Applied-Geotechnological-Solutions/Web_map_demo_data/blob/main/icons/Movement/2.wtruck.png?raw=true',
      crossOrigin: undefined,
      scale: [icon_size.value, icon_size.value],
            }),
          })
      
      

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

      vectorContext.setStyle(Boat_Clinic_style); //Boat_Clinic_Point.getStyle() alternative ?
      vectorContext.drawGeometry(Boat_Clinic_position);

      vectorContext.setStyle(Water_T_style);
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
        font-size: 3vw;
        letter-spacing: normal;
        font-family: Lato,sans-serif;
        font-weight: 900;
        }

.button {
color: rgba(255, 255, 255, 1);
font-size: 0.8vw;
font-family: Lato,sans-serif;
white-space: pre-wrap;
line-height: 1.5;
position:relative;
padding-bottom: 5px;
stroke-width: 1px;
-webkit-text-stroke-color: white;
font-weight:semi-bold;
text-decoration: none;
margin: 2vh;


}
.button:hover {
  
  color: rgb(255, 65, 36);
  box-shadow: 0 2.6vh 3vh rgb(2 2 2 / 40%);
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
