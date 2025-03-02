<template>
  <div>
    <h3 class="mb-0">{{ data.name }}</h3>
    <h5 class="subtitle-name-text">{{ markerData.name }}</h5>
    <div class="found-toggle subtitle-name-text ">
      <b-form-checkbox v-model="found" name="check-button" switch v-on:input="updateMarker(markerData.type, data.id, $event, data.locations, data.isCollectible)">Complete: </b-form-checkbox>
    </div>
    <hr class="bg-light mb-2 mt-0"/>
    <div class="d-flex">
      <!-- add a side bar for quest chain if there's multiple parts -->
      <div v-if="data.markerLink" class="popup-sidebar pr-3 text-center">
        <h5> Quest Chain </h5>
        <div v-for="link in data.markerLink" :key="link.label">
          <b-link v-on:click="changeMap(link.zone, link.marker, link.index)"> {{ link.label }} </b-link>
        </div>
      </div>

      <!-- core content -->
      <div class="popup-content pl-3 pr-3 mt-2 text-center">
        <!-- html content -->
        <p v-if="data.content" class="mt-0 mb-2" v-html="data.content" />
        <p v-if="locationData && locationData.content" class="mt-0 mb-2" v-html="locationData.content" />

        <!-- merchant module -->
        <wandering-merchant-detail v-if="markerData.type === 'merchant'" :merchant-id="data.id" :disambiguator="uuidv4()"></wandering-merchant-detail>

        <!-- affinity module -->
        <affinity-detail v-if="markerData.type === 'affinity'" :affinity-id="data.id" :disambiguator="uuidv4()"></affinity-detail>

        <!-- zone collection -->
        <div v-if="data.mapId" class="d-flex flex-wrap mt-3 mb-2 text-left" :set="playerData = getPlayerData(data.mapId)">
          <div v-for="(marker) in data.markers" :key="marker.type" class="zone-markers ml-3">
            <b-img width="18px" class="mb-1" :src="iconData[marker.type].icon" /> {{ iconData[marker.type].name }} ({{ playerData && playerData[marker.type]? playerData[marker.type].c : 0}}/{{marker.count}})
          </div>
        </div>

        <!-- image -->
        <b-button v-if="data.image" v-b-modal="markerData.type.toString()+data.id.toString()" variant="link" size="sm">
          <b-img width="128" :src="data.image"></b-img>
        </b-button>
        <b-modal v-if="data.image" :id="markerData.type.toString()+data.id.toString()" :hide-footer="true" :hide-header="true" body-bg-variant="dark">
          <b-img fluid :src="data.image"></b-img>
        </b-modal>

        <!-- location specific image -->
        <b-button v-if="locationData && locationData.image" v-b-modal="markerData.type.toString()+data.id.toString()+locationData.index.toString()" variant="link" size="sm">
          <b-img width="128" :src="locationData.image"></b-img>
        </b-button>
        <b-modal v-if="locationData && locationData.image" :id="markerData.type.toString()+data.id.toString()+locationData.index.toString()" :hide-footer="true" :hide-header="true" body-bg-variant="dark">
          <b-img fluid :src="locationData.image"></b-img>
        </b-modal>

        <!-- copy node location button -->
        <div v-if="data.mapId" class="text-center mb-2">
          <b-button size="sm" class="mr-2" v-on:click="copyLocation(mapId, data.id, index)">
            Copy Location
          </b-button>
          <b-button size="sm" v-on:click="changeMap(data.mapId)"> Enter </b-button>
        </div>
        <div v-else class="text-center mt-3 mb-1">
          <b-button size="sm" v-on:click="copyLocation(mapId, data.id, index)">
            Copy Location
          </b-button>
        </div>
      </div>

      <!-- add a side bar for other location if this can be done somewhere else -->
      <div v-if="data.locations" class="popup-sidebar-right pl-3 text-center">
        <h5> Other Location </h5>
        <div v-for="(location, lIndex) in data.locations" :key="lIndex">
          <b-link v-on:click="changeMap(location.zone, data.id, location.index)"> {{ 'Location ' + (lIndex + 1) }} </b-link>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import iconData from "@/assets/data/map/icon.json";
import WanderingMerchantDetail from "@/components/WanderingMerchantDetail";
import AffinityDetail from "@/components/AffinityDetail";

export default {
  name: "GameMapPopup",
  components: {AffinityDetail, WanderingMerchantDetail},
  props: ['data', 'markerData', 'locationData', 'isFound', 'index', "mapId"],
  data: function () {
    return {
      found: this.isFound,
      iconData: iconData
    }
  },
  methods: {
    copyLocation: function(mapId, markerId, indexId) {
      let url = location.protocol + '//' + location.host + location.pathname + '#' + this.$route.path + '?m=' + mapId + '&c=' + markerId;
      if (indexId >= 0) {
        url += '&i=' + indexId;
      }
      navigator.clipboard.writeText(url)
      this.$notify({group: 'copy', text: 'Copied location to clipboard.'})
    },
    updateMarker: function(type, id, newState, locations, isCollectible) {
      this.$emit('updateMarker', type, id, newState, locations, isCollectible);
    },
    getPlayerData: function(mapId) {
      if (localStorage.getItem(mapId + '_map')) {
        try {
          return JSON.parse(localStorage.getItem(mapId + '_map'));
        } catch (e) {
          localStorage.removeItem(mapId + '_map');
          return {};
        }
      }
    },
    updateFound: function(newState) {
      this.found = newState;
    }
  }
}
</script>

<style>
/* Complete Switch */
.found-toggle .custom-control .custom-control-label::before {
  left: 5rem !important;
}
.found-toggle .custom-control .custom-control-label::after {
  left: calc(5rem + 2px) !important;
}

.subtitle-name-text {
  color: rgba(255, 255, 255, 0.3);
}

.found-toggle {
  position: absolute;
  right: 3.5rem;
  top: 2.8rem;
  font-size: 1rem !important;
}

.popup-sidebar {
  border-right: 1.5px solid;
}

.popup-sidebar-right {
  border-left: 1.5px solid;
}

.popup-content {
  flex-grow: 1;
}

.zone-markers {
  flex: 40%;
}

.modal-content {
  border: 0 !important;
}
</style>