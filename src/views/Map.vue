<template>
  <div>
    <div class="title">ArcGis test</div>
    <div ref="mapContainer" class="map" id="map"></div>
    <!-- <div ref="map" class="map">
      <div ref="button"></div>
    </div>-->
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
        "esri/layers/TiledMapServiceLayer",
        "esri/layers/FeatureLayer",
        "esri/symbols/SimpleMarkerSymbol",
        "esri/symbols/SimpleLineSymbol",
        "esri/renderers/SimpleRenderer",
        "dojo/_base/Color"
      ],
      { url: baseUrl.apiUrl, css: baseUrl.cssUrl }
    )
      .then(
        ([
          Map,
          Tiled,
          FeatureLayer,
          SimpleMarkerSymbol,
          SimpleLineSymbol,
          SimpleRenderer,
          Color
        ]) => {
          // 创建一个底图
          const map = new Map(this.$refs.mapContainer, {
            basemap: "topo",
            center: [118.79647, 32.05838], //南京城区
            zoom: 10, // 当前地图缩放等级
            logo: false
          });

          map.on("load", () => {
            // 创建一个图层
            const baseLayer = new Tiled(baseUrl.baseMap);
            // 将图层添加到map中
            map.addLayer(baseLayer, 1);

            // 创建一个line用做marker的outline
            const layer2Outline = new SimpleLineSymbol({
              style: SimpleLineSymbol.STYLE_SOLID,
              color: new Color([255, 0, 0]),
              size: 1
            });
            // 创建一个marker
            const layer2Symbol = new SimpleMarkerSymbol({
              style: SimpleMarkerSymbol.STYLE_SQUARE,
              size: 10,
              outline: layer2Outline,
              color: new Color([0, 255, 0, 0.25])
            });
            //创建一个render
            const layer2Sr = new SimpleRenderer({
              symbol: layer2Symbol
            });
            // 创建新图层
            const layer = new FeatureLayer(baseUrl.layer2, {
              title: "layer2"
              // renderer: layer2Sr
            });
            // 将图层添加到map中
            map.addLayer(layer, 2);
          });
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
