<template>
  <div id="map">
    <div class="search-card">
      <div class="card-header">
        <span>数据查询</span>
        <el-button
          @click="toogleSearch()"
          style="float: right; padding: 3px 0"
          type="text"
        >{{toogleSearchBtn}}</el-button>
      </div>
      <div v-if="searchStatus" class="divider"></div>
      <el-form class="serach-form" label-position="left" :model="search" v-if="searchStatus">
        <el-form-item>
          <el-select v-model="search.layer" class="layer-item" placeholder="请选择图层">
            <el-option v-for="item in layers" :key="item.id" :label="item.title" :value="item.id"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item>
          <el-select v-model="search.fileds" class="layer-item" multiple placeholder="请选择查询的域">
            <el-option v-for="item in fileds" :key="item" :label="item" :value="item"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item>
          <el-input v-model="search.text" class="layer-item" placeholder="输入字段">
            <el-button id="searchBtn" slot="append" icon="el-icon-search"></el-button>
          </el-input>
        </el-form-item>
      </el-form>
    </div>

    <div ref="mapContainer" class="map"></div>
  </div>
</template>

<script>
import { loadModules } from "esri-loader";
import baseUrl from "../../mapconfig";

export default {
  name: "Map",
  data() {
    return {
      map: null,
      search: {
        layer: "",
        fileds: [],
        text: ""
      },
      searchStatus: true,
      searchResult: [],
      layers: []
    };
  },
  computed: {
    toogleSearchBtn: function() {
      return this.searchStatus === true ? "收起" : "展开";
    },
    fileds: function() {
      if (this.search.layer == null || this.search.layer === "") return [];
      return this.layers[parseInt(this.search.layer)].fileds;
    }
  },
  methods: {
    toogleSearch() {
      this.searchStatus = !this.searchStatus;
    }
    // findByAttr(search) {
    //   //-------------查询----------------//
    //   //属性查询
    //   loadModules([
    //     "esri/symbols/SimpleLineSymbol",
    //     "esri/symbols/SimpleFillSymbol"
    //   ])
    //     .then(
    //       ([FindTask, FindParameters, SimpleLineSymbol, SimpleFillSymbol]) => {
    //         const findParameters = new FindParameters();
    //         // const layerId = parseInt(this.search.layer);
    //         findParameters.returnGeometry = true;
    //         // findParameters.layerIds = [layerId];
    //         findParameters.layerIds = [
    //           "customLayer0",
    //           "customLayer1",
    //           "customLayer2",
    //           "customLayer3"
    //         ];
    //         findParameters.searchFields = this.search.fileds;
    //         findParameters.searchText = this.search.text;
    //         const findTask = new FindTask(baseUrl.baseMap);
    //         findTask.execute(findParameters).then(showResult);
    //         console.log("map:", this.map);
    //         function showResult(result) {
    //           console.log("result:", result);
    //           // console.log("layer1:", this.map.getLayer("layer1"));
    //           // //查询工具
    //           // this.map.graphics.clear();
    //           // //线符号
    //           // const lineSymbol = new SimpleLineSymbol(
    //           //   SimpleLineSymbol.STYLE_DASH,
    //           //   new dojo.Color([0, 0, 255]),
    //           //   3
    //           // );
    //           // //面符号
    //           // const fillSymbol = new SimpleFillSymbol(
    //           //   SimpleFillSymbol.STYLE_SOLID,
    //           //   lineSymbol
    //           // );
    //           // if (result.length == 0) {
    //           //   alert("未查询到任何信息");
    //           //   return;
    //           // } else {
    //           //   for (let i = 0; i < result.length; i++) {
    //           //     //获得图形graphic
    //           //     var graphic = result[i].feature;
    //           //     //赋予相应的符号
    //           //     graphic.setSymbol(lineSymbol);
    //           //     //将graphic添加到地图中，从而实现高亮效果
    //           //     this.map.graphics.add(graphic);
    //           //   }
    //           // }
    //         }
    //       }
    //     )
    //     .catch(err => {
    //       window.console.log(err);
    //     });
    // }
  },
  created() {
    this.layers = baseUrl.layers;
  },
  mounted() {
    loadModules(
      [
        "esri/map",
        "esri/layers/ArcGISDynamicMapServiceLayer",
        "esri/layers/FeatureLayer",
        "esri/tasks/FindTask",
        "esri/tasks/FindParameters",
        "dojo/domReady!",
        "dojo/query",
        "dojo/on",
        "esri/Color",
        "esri/symbols/SimpleLineSymbol",
        "esri/symbols/SimpleFillSymbol"
      ],
      {
        url: baseUrl.apiUrl,
        css: baseUrl.cssUrl
      }
    )
      .then(
        ([
          Map,
          ArcGISDynamicMapServiceLayer,
          FeatureLayer,
          FindParameters,
          FindTask,
          SimpleLineSymbol,
          SimpleFillSymbol,
          query,
          on,
          Color
        ]) => {
          // 创建一个底图
          this.map = new Map(this.$refs.mapContainer, {
            basemap: "topo",
            center: [118.99647, 31.50838], //南京城区
            zoom: 10, // 当前地图缩放等级
            logo: false
          });
          // 创建底图图层，并将图层添加到map中
          const baseLayer = new ArcGISDynamicMapServiceLayer(baseUrl.baseMap);
          this.map.addLayer(baseLayer);
          // 创建图层,并将图层添加到map中
          baseUrl.layers.forEach(async layer => {
            this.map.addLayer(
              await new FeatureLayer(layer.url, {
                title: layer.title,
                id: "customLayer" + layer.id
              }),
              layer.id
            );
          });

          //查询工具
          const findTask = new FindTask(baseUrl.baseMap);
          //属性查询
          const attrFinfd = function() {
            console.log("begin-find");
            const findParameters = new FindParameters();
            const layerId = parseInt(this.search.layer);
            findParameters.returnGeometry = true;
            findParameters.layerIds = [layerId];
            findParameters.layerIds = [
              "customLayer0",
              "customLayer1",
              "customLayer2",
              "customLayer3"
            ];
            findParameters.searchFields = this.search.fileds;
            findParameters.searchText = this.search.text;
            console.log("params:", findParameters);
            findTask.execute(findParameters, showResult);
          };
          //属性查询渲染函数
          const showResult = function(result) {
            console("result:", result);
            this.map.graphics.clear();
            //线符号
            var lineSymbol = new SimpleLineSymbol(
              SimpleLineSymbol.STYLE_DASH,
              new Color([0, 0, 255]),
              3
            );
            //面符号
            var fillSymbol = new SimpleFillSymbol(
              SimpleFillSymbol.STYLE_SOLID,
              lineSymbol
            );
            if (result.length == 0) {
              alert("未查询到任何信息");
              return;
            } else {
              for (var i = 0; i < result.length; i++) {
                //获得图形graphic
                var graphic = result[i].feature;
                //赋予相应的符号
                graphic.setSymbol(fillSymbol);
                //将graphic添加到地图中，从而实现高亮效果
                this.map.graphics.add(graphic);
              }
            }
          };

          const searchBtn = query("#searchBtn");
          
          on(searchBtn, "click", attrFinfd);
        }
      )
      .catch(err => {
        window.console.log(err);
      });
  }
};
</script>

<style scoped>
#map {
  height: 95vh;
  padding: 0;
  margin: 0;
}
.map {
  box-sizing: border-box;
  height: 100%;
}
.search-card {
  width: 250px;
  position: absolute;
  right: 20px;
  top: 50px;
  z-index: 10;
  padding: 8px;
  background-color: #fff;
  border: 1px solid #dcdfe6;
  border-radius: 5px;
}
.divider {
  display: block;
  width: 100%;
  margin: 10px 0 20px;
  height: 0px;
  border-bottom: 1px solid #ebeef5;
}
.serach-form {
  margin-bottom: 5px;
}
.layer-item {
  width: 250px;
}
</style>
