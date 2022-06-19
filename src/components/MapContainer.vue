<template>
  <div id="container"></div>
</template>

<script>
import AMapLoader from '@amap/amap-jsapi-loader';
import { getPositions } from "../data/data.js";
export default {
  name: 'MapContainer',
  data() {
    return {
      positions: [],
    }
  },
  beforeCreate() {
    AMapLoader.load({
      // 申请好的Web端开发者Key，首次调用 load 时必填
      key: 'a915f8e665dde88da8949555e35c5886',
      // 指定要加载的 JSAPI 的版本，缺省时默认为 1.4.15        
      version: "1.4.15",
      // 需要使用的的插件列表，如比例尺'AMap.Scale'等     
      plugins: ['AMap.CircleEditor'],
      AMapUI: {
        // 是否加载 AMapUI，缺省不加载
        version: "1.1", // AMapUI 缺省 1.1
        plugins: [], // 需要加载的 AMapUI ui插件
      },
      Loca: {
        // 是否加载 Loca， 缺省不加载
        version: "1.3.2", // Loca 版本，缺省 1.3.2
      },
    }).then(() => {
      this.$nextTick(() => this.initLoad());
    }).catch(e => {
      console.log(e);
    })
  },
  methods: {
    initLoad() {
      this.positions = getPositions()
      this.initMap();
    },
    initMap() {
      //设置地图容器id
      this.map = new AMap.Map("container", {
        //是否为3D地图模式
        viewMode: "2D",
        //初始化地图级别  
        zoom: 11,
        //初始化地图样式         
        mapStyle: 'amap://styles/darkblue',
        //地图中心点
        center: [121.48, 31.22],
        resizeEnable: true
      });
      this.setMarkers()
    },
    setMarkers() {
      let that = this;
      const styleObject = {
        url: "//a.amap.com/jsapi_demos/static/demo-center/icons/poi-marker-default.png",
        size: new AMap.Size(19, 33),
        anchor: new AMap.Pixel(0, 0) // 图标显示位置偏移量，基准点为图标左上角
      }
      const massMarks = new AMap.MassMarks(null, {
        zIndex: 50,  // 海量点图层叠加的顺序
        zooms: [4, 20],  // 在指定地图缩放级别范围内展示海量点图层
        style: styleObject
      });
      massMarks.setData(that.positions);
      massMarks.on('click', function (e) {
        const markItem = that.positions.find(position => position.keyId === e.data.keyId)
        if (!markItem?.onClick) {
          markItem.line = {}
          markItem.close = {}
          markItem.markerModal = {}
          that.setMarkerModal(e)
        }
      });
      massMarks.setMap(that.map);
    },
    setMarkerModal(targetEvent) {
      const markItem = this.positions.find(position => position.keyId === targetEvent.data.keyId)
      markItem.onClick = true;
      const text = new AMap.Text({
        text: `<div style="color: green" >
          <div class='mark-text' >${markItem.name}</div>
        </div>`,
        anchor: 'center', // 设置文本标记锚点
        draggable: true,
        cursor: 'pointer',
        style: {
          'padding': '.75rem 1.25rem',
          'margin-bottom': '1rem',
          'border-radius': '.25rem',
          'background-color': 'white',
          'width': '15rem',
          'border-width': 0,
          'text-align': 'center',
          'font-size': '20px',
          'color': '#000000'
        },
        position: [markItem.lnglat.lng + 0.1, markItem.lnglat.lat + 0.1]
      });
      text.setMap(this.map);
      markItem.markerModal = text
      const position = text.getPosition()
      this.showLine(position, targetEvent)
      text.on('dragging', (e) => {
        this.showLine(e.target.De.position, targetEvent)
      });

    },
    showLine(position, targetEvent) {
      const markItem = this.positions.find(position => position.keyId === targetEvent.data.keyId)
      this.map.remove(markItem?.line);
      this.map.remove(markItem?.close);
      const path = [
        [targetEvent.data.lnglat.lng + 0.01, targetEvent.data.lnglat.lat - 0.01],
        [position.lng, targetEvent.data.lnglat.lat - 0.01],
        [position.lng, position.lat],
      ];
      const polyline = new AMap.Polyline({
        path: path,
        borderWeight: 1, // 线条宽度，默认为 1
        strokeColor: '#ffffff', // 线条颜色
        lineJoin: 'round', // 折线拐点连接处样式
        strokeStyle: 'dashed',
        zIndex: 50,
      })
      markItem.line = polyline
      this.setCloseIcon([position.lng, position.lat], targetEvent)
      this.map.add(polyline);
    },
    setCloseIcon(position, targetEvent) {
      let that = this
      const markItem = this.positions.find(position => position.keyId === targetEvent.data.keyId)
      const marker = new AMap.Marker({
        position: position,
        offset: new AMap.Pixel(100, -20),
        icon: 'https://github.com/KiligFei/machineroom/blob/master/public/models/close-icon.png?raw=true',
      });
      marker.on('click', function () {
        that.map.remove(markItem?.line);
        that.map.remove(markItem?.close);
        that.map.remove(markItem?.markerModal);
        markItem.onClick = false;
      });
      markItem.close = marker
      marker.setMap(that.map);
    },
  },

}
</script>

<style  scoped >
#container {
  padding: 0px;
  margin: 0px;
  width: 100%;
  height: 800px;
}
</style>
