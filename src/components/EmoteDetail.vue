<template>
  <div v-if="emoteData && itemData">
    <div :class="['d-flex', 'flex-column']">
      <div class="d-flex">
        <item-icon :data="itemData" :id="emoteId + uuidv4()" :key="itemData.id" :hide_popover="emoteData.acquisitionType === 'default'" size="0.9"/>
        <div class="d-flex flex-column ml-2 text-left flex-fill">
          <div class="d-flex">
            <h6 class="m-0">{{emoteData.name}}</h6>
          </div>
          <p v-if="points"  class="m-0 points"> {{points}} Affinity Points</p>
          <p class="mb-1 mt-auto"> {{emoteData.description}}</p>
        </div>
      </div>
      <hr class="tooltiphr">
      <div v-if="emoteData.acquisitionType !== 'default'" class="text-left mt-1 d-flex align-items-center">
        <h6 class="m-0"> Acquisition Method </h6>
        <div class="ml-auto mr-4 subtitle-name-text acquired-font">
          <b-form-checkbox v-if="emoteData.acquisitionType !== 'default'" v-model="playerEmoteData[emoteId]" name="check-button" switch v-on:input="updateCollectible('emote', emoteId, $event, emoteData.acquisitionContinentId, emoteData.acquisitionLocation);">Acquired: </b-form-checkbox>
        </div>
      </div>
      <div  v-if="emoteData.acquisitionLink" >
        <b-link class="d-inline-block" :href="emoteData.acquisitionLink">
          <b-img v-if="emoteData.acquisitionType && emoteData.acquisitionType!=='exchange' && emoteData.acquisitionType!=='adventures_tome'" :src="'./images/icons/' + emoteData.acquisitionType + '.png'" class="pb-1"/>
          <span :class="['d-inline-block', 'acquisition-text', emoteData.acquisitionType ? emoteData.acquisitionType : '', 'ml-1']">
            {{emoteData.acquisition}}
          </span>
        </b-link>
        <div class="text-white" v-if="emoteData.additionalInfo">
          {{ emoteData.additionalInfo }}
        </div>
      </div>
      <div v-else>
        <b-img v-if="emoteData.acquisitionType !== 'default'" :src="'./images/icons/' + emoteData.acquisitionType + '.png'" class="pb-1"/>
        <span :class="['d-inline-block', 'acquisition-text', emoteData.acquisitionType ? emoteData.acquisitionType : '', 'ml-1']">
          {{emoteData.acquisition}}
        </span>
        <div class="text-white" v-if="emoteData.additionalInfo">
          {{ emoteData.additionalInfo }}
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import ItemIcon from "@/components/ItemIcon";
import '@/assets/css/main.css';

export default {
  name: "EmoteDetail",
  props: ["emoteId", "points"],
  components: {ItemIcon},
  data: function() {
    return {
      found: false,
      emoteData: null,
      itemData: null,
      playerEmoteData: null
    }
  },
  async created() {
    this.emoteData = await import("@/assets/data/emotes/" + this.emoteId + ".json");
    this.itemData = await import("@/assets/data/items/" + this.emoteId + ".json");

    this.playerEmoteData = {};
    if (localStorage.getItem('emote_collectibles')) {
      try {
        let playerCollectibles = JSON.parse(localStorage.getItem('emote_collectibles'));

        if (playerCollectibles[this.emoteData.acquisitionContinentId]) {
          this.playerEmoteData = playerCollectibles[this.emoteData.acquisitionContinentId];
        }
      } catch (e) {
        localStorage.removeItem('emote_collectibles');
      }
    }

    this.$forceUpdate()
  }
}
</script>

<style>

/* Acquire Switch */
.subtitle-name-text .custom-control-label::before {
  left: 4.5rem !important;
}
.subtitle-name-text .custom-control-label::after {
  left: calc(4.5rem + 2px) !important;
}
</style>

<style scoped>
.acquisition-method {
  height: 18px;
}

.acquired-font {
  font-size: 1rem !important;
}

.points {
  color: #19a7e5;
}
</style>