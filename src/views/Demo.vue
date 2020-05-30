<template>
  <div>
    <div class="title">ArcGis test</div>
    <div ref="mapContainer" class="map"></div>
  </div>
</template>

<script>
import { loadModules } from "esri-loader";
import baseUrl from "../../mapconfig";

export default {
  name: "Map",
  data() {
    return {};
  },
  mounted() {
    loadModules(
      [
        "esri/map",
        "esri/layers/ArcGISDynamicMapServiceLayer",
        "esri/layers/FeatureLayer",
        "dojo/domReady!"
      ],
      {
        url: baseUrl.apiUrl,
        css: baseUrl.cssUrl
      }
    )
      .then(
        ([Map, ArcGISDynamicMapServiceLayer, FeatureLayer]) => {
       
          // 创建一个底图
          const map = new Map(this.$refs.mapContainer, {
            basemap: "topo",
            center: [118.79647, 32.05838], //南京城区
            zoom: 10, // 当前地图缩放等级
            logo: false
          });
        //   map.spatialReference = { wkid: 4326 };
          // 创建图层
          const baseLayer = new ArcGISDynamicMapServiceLayer(baseUrl.baseMap);
          console.log(baseLayer);
          // 将图层添加到map中
          map.addLayer(baseLayer, 1);
          // console.log("map", map.getLayers());
          // // 创建新图层
          const layer = new FeatureLayer(baseUrl.layer2, {
            title: "layer2"
          });
          // 将图层添加到map中
          map.addLayer(layer, 2);
        }
      )
      .catch(err => {
        window.console.log(err);
      });
  }
};
</script>

<style scoped>
.title {
  text-align: center;
}
.map {
  height: 90vh;
  border: 1px solid red;
}
</style>
