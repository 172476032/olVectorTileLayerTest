<template>
  <div>
    <div id="map">
    </div>
  </div>

</template>

<script>
import "ol/ol.css";
import Map from "ol/Map";
import View from "ol/View";
import OSM from "ol/source/OSM";
import TileLayer from "ol/layer/Tile";
import VectorTileLayer from "ol/layer/VectorTile";
import VectorTile from "ol/source/VectorTile";
import MVT from "ol/format/MVT";
import XYZ from "ol/source/XYZ";
import TileWMS from "ol/source/TileWMS";
import RenderFeature from "ol/render/Feature";
import { transform } from "ol/proj";
import Stroke from "ol/style/Stroke";
import Style from "ol/style/Style";
import Fill from "ol/style/Fill";
import Text from "ol/style/Text";
import Icon from "ol/style/Icon";
import { pointerMove, click } from "ol/events/condition";
import Select from "ol/interaction/Select";

export default {
  data() {
    return {
      map: null,
      treeLayers: [
        {
          id: "10101",
          name: "水库",
          type: "VECTORTILE",
          geoType: "point",
          layerName: "cjliuyushuiku",
          visible: true,
          layerUrl:
            "/geoserver166/gwc/service/tms/1.0.0/test:cjliuyushuiku@EPSG%3A900913@pbf/{z}/{x}/{-y}.pbf",
          vectorConfig: {
            zoom: 0,
            labelField: "测站名",
            minZoomStyle: (text, zoom) => {
              return new Style({
                image: new Icon({
                  src: shuiku,
                  scale: 0.3
                })
              });
            },
            maxZoomStyle: text => {
              return new Style({
                image: new Icon({
                  src: shuiku,
                  scale: 0.5
                }),
                text: new Text({
                  text: text,
                  offsetY: -20,
                  offsetX: 20,
                  font: "5px sans-serif",
                  padding: [5, 8, 5, 8],
                  fill: new Fill({
                    color: "#00B5FF"
                  }),
                  stroke: new Stroke({
                    color: "#ffff",
                    width: 1
                  })
                })
              });
            },
            hoverStyle: text => {
              return new Style({
                image: new Icon({
                  src: shuiku,
                  scale: 0.6
                }),
                text: new Text({
                  text: text,
                  offsetY: -20,
                  offsetX: 20,
                  font: "bold 13px sans-serif",
                  padding: [5, 8, 5, 8],
                  fill: new Fill({
                    color: "#00B5FF"
                  }),
                  stroke: new Stroke({
                    color: "#ffff",
                    width: 0
                  })
                })
              });
            }
          }
        }
      ]
    };
  },
  components: {},
  mounted() {
    this.init();
  },
  computed: {},
  methods: {
    init() {
      //地图初始化
      this.map = new Map({
        view: new View({
          center: [11815338.577971682, 3558032.468689678], // [103.91678460032786, 26.735038084423355], // 金沙江流域
          zoom: 5,
          minZoom: 1,
          maxZoom: 18,
          projection: "EPSG:3857"
        }),
        layers: [
          new TileLayer({
            source: new OSM()
          })
        ],
        target: "map"
      });
      this.addLayers(this.map);
    },
    addLayers(map) {
      let layers = [];
      this.treeLayers.forEach(v => {
        console.log("v: ", v);
        let layer = new VectorTileLayer({
          source: new VectorTile({
            format: new MVT(),
            url: v.layerUrl
          }),
          preload: 6,
          // style: f => {
          //   // console.log("f: ", f);
          //   let props = f.getProperties(),
          //     zoom = this.map.getView().getZoom();
          //   if (zoom >= v.vectorConfig.zoom) {
          //     return v.vectorConfig.maxZoomStyle(
          //       zoom,
          //       f,
          //       this.selection,
          //       props
          //     );
          //   } else {
          //     return v.vectorConfig.minZoomStyle(
          //       zoom,
          //       f,
          //       this.selection,
          //       props
          //     );
          //   }
          // },
          zIndex: 6000
        });
        layer.setVisible(v.visible);
        this.setLayerIndex(v.geoType, layer);
        map.addLayer(layer);
        layers.push(layer);
      });
      this.addInteractions(map, layers);
    },
    addInteractions(map, layers) {
      let vtPointMoveEvt = new Select({
        layers: layers,
        condition: pointerMove,
        hitTolerance: 5
      });
      vtPointMoveEvt.on("select", e => {
        console.log("pointMove: ", e);
      });
      map.addInteraction(vtPointMoveEvt);
      let vtClick = new Select({
        layers: layers,
        condition: click,
        hitTolerance: 5
      });
      vtClick.on("select", e => {
        console.log("click: ", e);
      });
      map.addInteraction(vtClick);
    },

    setLayerIndex(type, layer) {
      let zIndex = [200, 400, 600, 800];
      if (type == "point") {
        layer.setZIndex(zIndex[3]);
      } else if (type == "line") {
        layer.setZIndex(zIndex[2]);
      } else if (type == "polygon") {
        layer.setZIndex(zIndex[1]);
      } else if (type == "buttom") {
        layer.setZIndex(zIndex[0]);
      }
    }
  },
  destroyed() {}
};
</script>

<style lang="scss"  >
#map {
  width: 100%;
  height: 100%;
}
</style>
