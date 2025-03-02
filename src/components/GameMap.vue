<template>
  <div id="map">
    <l-map v-if="dataReady"
        ref="map"
        :zoom="zoomLevel"
        :min-zoom="minZoom"
        :max-zoom="maxZoom"
        :max-bounds="maxBounds"
        :crs="crs"
        :center="center"
        @click="printLocation"
        @contextmenu.native="contextMenu($event)"
    >
      <!-- This is the map -->
      <l-tile-layer
          v-if="mapUrl"
          :url="mapUrl"
          :no-wrap="true"
          :attribution="attribution"
          :options="mapOptions"
      />

      <div v-if="mapData">
        <!-- List of markers collection markers-->
        <div v-for="(marker) in mapData.markers" :key="marker.type">
          <div v-if="iconData[marker.type] && !marker.world">
            <div v-for="(data) in marker.data" :key="data.id">
              <!-- single marker -->
              <l-marker v-if="!data.locations && (!playerData[marker.type] || !playerData[marker.type].h)" :lat-lng="data" :ref="data.id">
                <l-icon :icon-anchor="iconData[marker.type].anchor">
                  <b-img v-if="iconData[marker.type].icon" fluid :src="iconData[marker.type].icon" :class="[playerData[marker.type] && playerData[marker.type][data.id] ? 'found' : '', playerData[marker.type] && playerData[marker.type].h ? 'hidden' : '' ]" />
                  <p :style="'color:' + iconData[marker.type].color + '!important;'" :class="[playerData[marker.type] && playerData[marker.type][data.id] ? 'found' : '', playerData[marker.type] && playerData[marker.type].h ? 'hidden' : '', 'icon-display-name']">
                    <span v-html="data.display" />
                  </p>
                </l-icon>
                <l-popup class="text-light">
                  <game-map-popup :data="data" :marker-data="iconData[marker.type]" :is-found="playerData[marker.type] && playerData[marker.type][data.id]" :map-id="mapData.id" @updateMarker="updateMarker" :key="data.id" :ref="location + '_' + data.id"/>
                </l-popup>
              </l-marker>

              <!-- multiple marker -->
              <div v-else-if="data.locations && (!playerData[marker.type] || !playerData[marker.type].h)" v-for="(location, lIndex) in data.locations" :key="lIndex">
                <l-marker v-if="location.zone === mapData.id"  :lat-lng="location" :ref="data.id">
                  <l-icon :icon-anchor="iconData[marker.type].anchor">
                    <b-img v-if="iconData[marker.type].icon" fluid :src="iconData[marker.type].icon" :class="[playerData[marker.type] && playerData[marker.type][data.id] ? 'found' : '', playerData[marker.type] && playerData[marker.type].h ? 'hidden' : '' ]" />
                    <p :style="'color:' + iconData[marker.type].color + '!important;'" :class="[playerData[marker.type] && playerData[marker.type][data.id] ? 'found' : '', playerData[marker.type] && playerData[marker.type].h ? 'hidden' : '', 'icon-display-name']">
                      <span v-html="data.display" />
                    </p>
                  </l-icon>
                  <l-popup class="text-light">
                    <game-map-popup :location-data="location" :data="data" :marker-data="iconData[marker.type]" :is-found="playerData[marker.type] && playerData[marker.type][data.id]" :map-id="mapData.id" :index="lIndex" @updateMarker="updateMarker" :key="data.id + lIndex.toString()" :ref="location.zone + '_' + data.id"/>
                  </l-popup>
                </l-marker>
              </div>
            </div>
          </div>

          <!-- List of world map markers-->
          <div v-if="marker.world && (!playerData[marker.type] || !playerData[marker.type].h)">
            <l-marker v-for="(data) in marker.data" :lat-lng="data" :key="data.id" :ref="data.id">
              <l-icon v-if="data.icon" :icon-anchor="data.anchor">
                <b-img fluid :src="data.icon" :class="[playerData[marker.type] && playerData[marker.type][data.id] ? 'found' : '', playerData[marker.type] && playerData[marker.type].h ? 'hidden' : '' ]" />
                <p :style="'color:' + marker.color + '!important;'" :class="[playerData[marker.type] && playerData[marker.type][data.id] ? 'found' : '', playerData[marker.type] && playerData[marker.type].h ? 'hidden' : '', 'icon-display-name']">
                  <span v-html="data.display" />
                </p>
              </l-icon>
              <l-popup class="text-light">
                <game-map-popup :data="data" :marker-data="marker" :is-found="playerData[marker.type] && playerData[marker.type][data.id]" @updateMarker="updateMarker" :map-id="mapData.id" :key="data.id" />
              </l-popup>
            </l-marker>
          </div>
        </div>
      </div>

    </l-map>

    <!-- Side bar -->
    <div v-if="mapData" id="menu-control">
      <b-button squared v-b-toggle.sidebar-right class="float-right">Menu</b-button>
      <b-sidebar id="sidebar-right" :title="this.mapData.name" right shadow bg-variant="dark" text-variant="light">
        <div class="px-3 py-2 mb-5">
          <div v-for="marker in mapData.markers" :key="marker.type" :class="playerData[marker.type] && playerData[marker.type].h ? 'found' : ''">
            <b-button v-if="iconData[marker.type].toggleable" squared switch class="container-fluid text-left mt-1" v-on:click="toggleMarker(marker.type)">
              <b-img width="18px" class="mr-2 mb-1" :src="iconData[marker.type].icon" /> {{ iconData[marker.type].name }} ({{playerData[marker.type]? playerData[marker.type].c : 0}}/{{marker.data.length}})
            </b-button>
            <b-button v-if="marker.world && marker.toggleable" squared switch class="container-fluid text-left mt-1" v-on:click="toggleMarker(marker.type)">
              <b-img width="18px" class="mr-2 mb-1" :src="marker.icon" /> {{ marker.name }} ({{playerData[marker.type]? playerData[marker.type].c : 0}}/{{marker.data.length}})
            </b-button>
          </div>
        </div>
        <div v-if="this.location != 'world'">
          <b-button squared class="container-fluid text-center fixed-bottom" v-on:click="changeMap('world')">
            Go to World Map
          </b-button>
        </div>
      </b-sidebar>
    </div>
  </div>
</template>

<script>

import { CRS, Icon, latLngBounds } from 'leaflet'
import { LMap, LTileLayer, LMarker, LIcon, LPopup } from 'vue2-leaflet'
import GameMapPopup from "@/components/GameMapPopup";
import iconData from "@/assets/data/map/icon.json";

// Leaflet/Webpack bug workaround - https://github.com/Leaflet/Leaflet/issues/4968
delete Icon.Default.prototype._getIconUrl

Icon.Default.mergeOptions({
  iconRetinaUrl: require('leaflet/dist/images/marker-icon-2x.png'),
  iconUrl: require('leaflet/dist/images/marker-icon.png'),
  shadowUrl: require('leaflet/dist/images/marker-shadow.png')
})

export default {
  name: "GameMap",
  components: {
    GameMapPopup,
    LMap,
    LTileLayer,
    LMarker,
    LIcon,
    LPopup
  },
  data: function () {
    return {
      dataReady: false,
      lastOpenedPopup: null,
      lastOpenedPopupIndex: null,
      lastOpenedPopupOffset: 0,
      location: this.$route.query.m,
      mapData: null,
      iconData: iconData,
      playerData: {},
      center: [-128, 128],
      attribution: "Created By Pokachi",
      mapUrl: null,
      minZoom: 1,
      maxZoom: 3,
      zoomLevel: 1,
      maxBounds: latLngBounds([[256, -256], [-512, 512]]),
      mapOptions: {
        bounds: latLngBounds([[0, 0], [-256, 256]])
      },
      popupOptions: {
        maxWidth: 400
      },
      crs: CRS.Simple,
    }
  },
  created() {
    this.importData()
  },
  methods: {
    saveData(key, data) {
      localStorage.setItem(key + '_map', JSON.stringify(data));
    },
    contextMenu(e) {
      e.preventDefault();
      if(this.mapData) {
        this.$router.push({query: {m: this.mapData.parent}}).catch(()=>{});
      }
    },
    printLocation(event) {
      var coord = event.latlng;
      var lat = coord.lat;
      var lng = coord.lng;
      console.log("You clicked the map at latitude: " + lat.toFixed(2) + " and longitude: " + lng.toFixed(2));
    },
    toggleMarker(type) {
      if (!this.playerData[type]) {
        this.playerData[type] = {};
        this.playerData[type].c = 0;
        this.playerData[type].h = false;
      }
      this.playerData[type].h = !this.playerData[type].h;

      this.$forceUpdate();
      this.saveData(this.location, this.playerData);
    },
    updateMarker(type, id, newState, locations, isCollectible) {
      if (isCollectible) {
        this.updateCollectible(type, id, newState, this.mapData.continentId, null);
      }

      if (!this.playerData[type]) {
        this.playerData[type] = {};
        this.playerData[type].c = 0;
        this.playerData[type].h = false;
      }
      if (newState === this.playerData[type][id]) {
        return;
      }

      this.playerData[type][id] = newState;
      if (newState) {
        this.playerData[type].c++;
      } else {
        this.playerData[type].c--;
      }

      if(locations) {
        locations.forEach(location => {
          if (location.zone !== this.location) {
            let tempData = {};
            if (localStorage.getItem(location.zone + '_map')) {
              try {
                tempData = JSON.parse(localStorage.getItem(location.zone + '_map'));
              } catch (e) {
                localStorage.removeItem(location.zone + '_map');
              }
            }

            if (!tempData[type]) {
              tempData[type] = {};
              tempData[type].c = 0;
              tempData[type].h = false;
            }
            if (newState === tempData[type][id]) {
              return;
            }
            tempData[type][id] = newState;
            if (newState) {
              tempData[type].c++;
            } else {
              tempData[type].c--;
            }
            this.saveData(location.zone, tempData);
          }
          if (location.zone === this.location) {
            for (const ref of this.$refs[location.zone + '_' + id]) {
              ref.updateFound(newState);
            }

            if (this.mapData.adjacentZones) {
              for (const adjZone of this.mapData.adjacentZones) {
                this.$refs[location.zone + '_' + adjZone][0].$forceUpdate();
              }
            }
          }
        });
      }

      this.$forceUpdate();
      this.saveData(this.location, this.playerData);
    },
    async importData() {
      try {
        this.mapData = await import("@/assets/data/map/" + this.location + ".json");
      } catch (e) {
        this.location = "world";
        this.mapData = await import("@/assets/data/map/world.json");
      }
      this.mapUrl = './images/map/' + this.location + '/{z}/{x}/{y}.png';

      this.playerData = {};
      if (localStorage.getItem(this.location + '_map')) {
        try {
          this.playerData = JSON.parse(localStorage.getItem(this.location + '_map'));
        } catch (e) {
          localStorage.removeItem(this.location + '_map');
        }
      }

      if(this.location === "world") {
        this.maxBounds = latLngBounds([[-32, 0], [-224, 256]]);
      } else {
        this.maxBounds = latLngBounds([[512 / (this.mapData.minZoom), -776 / (this.mapData.minZoom)], [-512 / (this.mapData.minZoom) - 256, 776 / (this.mapData.minZoom) + 256]]);
      }
      this.minZoom = this.mapData.minZoom;
      this.maxZoom = this.mapData.maxZoom;
      this.zoomLevel = this.mapData.minZoom;
      if (this.$route.query.c && this.$route.query.i) {
        this.mapData.markers.forEach(marker => {
          marker.data.forEach(data => {
            if (data.id === this.$route.query.c) {
              try {
                this.center = [data.locations[this.$route.query.i].lat, data.locations[this.$route.query.i].lng];
                this.zoomLevel = this.mapData.maxZoom;
                this.lastOpenedPopupOffset = data.locations[this.$route.query.i].indexOffset;
              } catch (e) {
                this.center = [-128, 128];
              }
            }
          });
        });
      }
      else if (this.$route.query.c) {
        this.mapData.markers.forEach(marker => {
          marker.data.forEach(data => {
            if (data.id === this.$route.query.c) {
              this.center = [data.lat, data.lng];
              this.zoomLevel = this.mapData.maxZoom;
            }
          });
        });
      }
      this.dataReady = true;
    }
  },
  watch: {
    $route () {
      this.dataReady = false;
      if(this.$route.query.m) {
        this.location = this.$route.query.m;
      }
      this.importData()
      this.$forceUpdate()
    }
  },
  updated() {
    this.$nextTick(()=> {
      try {
        if ((this.$route.query.i && this.$route.query.c && this.lastOpenedPopup !== this.$route.query.c) || (this.$route.query.i && this.$route.query.i !== this.lastOpenedPopupIndex)) {
          this.$refs[this.$route.query.c][parseInt(this.$route.query.i) + this.lastOpenedPopupOffset].mapObject.openPopup();
          this.lastOpenedPopupIndex = this.$route.query.i;
          this.lastOpenedPopup = this.$route.query.c;
        }
        else if(this.$route.query.c && this.lastOpenedPopup !== this.$route.query.c) {
          this.$refs[this.$route.query.c][0].mapObject.openPopup();
          this.lastOpenedPopup = this.$route.query.c;
        }
      } catch (e) {
        // it's okay
      }
    })
  }
}
</script>

<style>
@import "../../node_modules/leaflet/dist/leaflet.css";

/* Hide close button */
.leaflet-popup-close-button {
  display: none !important;
}

/* map */
#map {
  height: calc(100vh - 56px);
  width: 100%;
}

/* Side Bar */
#sidebar-right {
  height: calc(60vh - 56px);
  top: 100px;
}

#menu-control {
  position: absolute;
  top:100px;
  right:0vh;
  z-index: 999;
}

.leaflet-container {
  background: #2d2d2d !important;
}

.icon-display-name {
  width: max-content;
  font-size: 14px;
  text-shadow: -1px 0 black, 0 1px black, 1px 0 black, 0 -1px black;
}

.leaflet-popup-tip {
  background: #3f3f3f !important;
}

.leaflet-popup-content-wrapper {
  background: #3f3f3f !important;
  white-space: pre-line;
}

.leaflet-popup-content {
  min-width: 400px;
}

.hidden {
  opacity: 0;
}
</style>
