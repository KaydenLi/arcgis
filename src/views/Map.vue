<template>
  <div>
    <div class="title">ArcGis test</div>
    <div ref="mapContainer" class="map"></div>
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
      ["esri/Map", "esri/layers/MapImageLayer", "esri/views/MapView"],
      { url: baseUrl.url }
    )
      .then(([Map, MapImageLayer, MapView]) => {
        // 创建一个图层
        const layer = new MapImageLayer({
          url: baseUrl.layer2,
          title: "BaseLayer"
        });
        // 创建一个底图
        const map = new Map({
          basemap: "topo" // 底图类型
        });
        map.add(layer);
        // 创建一个视图
        const view = new MapView({
          container: this.$refs.mapContainer, // 视图的容器
          map: map, // Map的实例放入视图中
          center: [118.79647, 32.05838], //南京城区
          zoom: 10 // 当前地图缩放等级
        });
        window.console.log(view);
      })
      .catch(err => {
        window.console.log(err);
      });
  }
  // mounted: function() {
  //   loadModules(["esri/map", "esri/dijit/HomeButton"], { url: baseUrl.url })
  //     .then(([Map, HomeButton]) => {
  //       // create map with the given options
  //       const map = new Map(this.$refs.map, {
  //         center: [-56.049, 38.485],
  //         zoom: 3,
  //         basemap: "gray"
  //       });

  //       // initialize the locate button
  //       // const button = new HomeButton(
  //       //   {
  //       //     map: map
  //       //   },
  //       //   this.$refs.button
  //       // );
  //       // button.startup();
  //     })
  //     .catch(err => {
  //       // handle any script or module loading errors
  //       console.error(err);
  //     });
  // }
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
