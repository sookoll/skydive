<template>
  <div id="map"></div>
  <div id="panel" v-if="list.length">
    <div v-for="item in list"
      :key="item.name"
      :class="item.layer.getVisible() ? 'item active' : 'item'"
      @click="item.layer.setVisible(!item.layer.getVisible())">{{ item.name }}</div>
  </div>
</template>

<script>
/**
 * @module examples.kml
 */
import OLCesium from 'olcs/OLCesium'
import { transform } from 'ol/proj'
import olView from 'ol/View'
import { defaults as defaultControls } from 'ol/control'
import OSM from 'ol/source/OSM'
import VectorSource from 'ol/source/Vector'
import {Stroke, Style} from 'ol/style'
import {Tile as TileLayer, Vector as VectorLayer} from 'ol/layer'
import olMap from 'ol/Map'
import GPX from 'ol/format/GPX'
import randomColor from 'randomcolor'

//window.Cesium.Ion.defaultAccessToken = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJqdGkiOiI0MzAyNzUyYi0zY2QxLTQxZDItODRkOS1hNTA3MDU3ZTBiMDUiLCJpZCI6MjU0MSwiaWF0IjoxNTMzNjI1MTYwfQ.oHn1SUWJa12esu7XUUtEoc1BbEbuZpRocLetw6M6_AA';

export default {
  name: 'Map',
  data() {
    return {
      center: [24.71728, 58.89001],
      zoom: 13,
      files: [
        '20200801.GPX',
        '20200801_1.GPX',
        '20200801_2.GPX',
        '20200802_1.GPX',
        '20200802_2.GPX',
        '20200906.GPX',
        '20200918.GPX',
        '20200920.GPX'
      ],
      list: []
    }
  },
  mounted() {
    const layers = []
    const list = []
    layers.push(new TileLayer({
      source: new OSM()
    }))
    this.files.forEach(item => {
      const color = randomColor({ luminosity: 'dark' })
      const style = new Style({
        stroke: new Stroke({
          color: color,
          width: 1
        })
      })
      const layer = new VectorLayer({
        source: new VectorSource({
          url: item,
          format: new GPX()
        }),
        style: style
      })
      list.push({
        layer: layer,
        color: color,
        name: item
      })
      layers.push(layer)
    })

    this.list = list

    const ol2d = new olMap({
      layers: layers,
      controls: defaultControls({ attribution: false }),
      target: 'map',
      view: new olView({
        center: transform(this.center, 'EPSG:4326', 'EPSG:3857'),
        zoom: this.zoom
      })
    })

    const ol3d = new OLCesium({map: ol2d})

    ol3d.setEnabled(true)

    const scene = ol3d.getCesiumScene()
    const camera = scene.camera
    console.log(camera)

    camera.lookUp(1.3)
    camera.moveDown(2500)



  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
#map {
  width: 60%;
  height: 100%;
}
#panel {
  position: absolute;
  width: 40%;
  height: 100%;
  right: 0;
  top: 0;
}
.item {
  padding: 1rem;
  border-bottom: 1px solid #eee
}
.item.active {
  font-weight: bold;
}
#map .cesium-credit-logoContainer,
#map .cesium-credit-textContainer {
  display: none!important
}
</style>
