<template>
  <div class="border icon-border">
    <div :id="id" class="position-relative fit-content">
      <b-img :style="['background-image: url(./images/items/background/icon_grade_' + data.grade + '.png)', size ? 'width: ' + 64 * size + 'px' : '']" class="item-icon" :src="data.image" />
      <b-img v-if="data.subIcon" class="item-sub-icon" :src="'./images/items/subicon/icon_' + data.subIcon + '.png'" :style="size ? 'width: ' + 28 * size + 'px' : ''"/>
      <span v-if="count" class="item-count"> {{count}} </span>
      <span v-if="cost" class="item-cost"> {{cost}} </span>
    </div>
    <div :id="'popover-container-'+id"></div>
    <b-popover v-if="!hide_popover" custom-class="item-popover text-left" :target="id" triggers="hover" :container="'popover-container-'+id">
      <h4 :class="['font-grade-' + data.grade, 'title', 'mb-0']"> {{data.name}} </h4>
      <hr class="bg-light mb-0 mt-0">
      <div v-if="data.type !== 'Card' && data.type !== 'Ship Crew'" :class="['d-flex', 'align-middle', 'text-left', 'pl-3', 'item-icon-popup' + data.grade, 'mb-2']">
        <div style="width: 64px; height: 64px" class="mt-2 mb-2">
          <b-img :src="data.image"/>
          <b-img v-if="data.subIcon" class="item-sub-icon-popup" :src="'./images/items/subicon/icon_' + data.subIcon + '.png'" />
        </div>
        <div class="fit-content ml-3">
          <div :class="['font-grade-' + data.grade, 'item-type']"> {{ data.type}} </div>
          <div v-if="data.cooldown"> {{ 'Cooldown: ' + data.cooldown}} </div>
        </div>
      </div>
      <div v-else-if="data.type === 'Card'">
        <b-img :style="'background-image: url(./images/items/card/' + data.id + '.png)'" :class="['item-card']" :src="'./images/items/card/frame/frame_grade_' + data.grade + '.png'" />
      </div>
      <div v-else >
        <b-img :style="['background-image: url(./images/items/background/icon_grade_' + data.grade + '.png)', 'height: 178px', 'width: 85px']" :class="['item-crew']" :src="'./images/items/crew/' + data.id + '.png'" />
      </div>
      <p v-if="data.bound" v-html="data.bound" class="text-left pl-3"/>
      <p v-html="data.description" class="text-left pl-3 pr-3 description"></p>
      <p v-if="data.destruction" v-html="data.destruction" class="text-left pl-3" />
    </b-popover>
  </div>
</template>

<script>
export default {
  name: "ItemIcon",
  props: ['data', 'showName', 'cost', 'count', 'id', 'size', 'hide_popover']
}
</script>

<style>

/* tooltip background color */
.item-popover .popover-body {
  background-color: #2d2d2d;
  box-shadow: 0 3px 14px rgb(0 0 0 / 40%);
  padding-left: 0 !important;
  padding-right: 0 !important;
  padding-top: 0 !important;
  min-width: 400px;
  color: white;
  text-align: center;
}
/* arrrow color */
.item-popover .arrow {
  display: none !important;
}
</style>

<style scoped>

.icon-border {
  border-color: #ffffff80 !important;
}

.item-popover {
  border: none;
}

.title {
  background-color: black;
  width: 100%;
  padding-top: 0.75rem;
  padding-bottom: 1rem;
}

.description {
  color: #d2ba72;
}

.font-grade-0 {
  color: white;
}
.item-icon-popup0 {
  background-image: radial-gradient(ellipse at bottom,
  #ffffff40 0,
  transparent 70%);
  background-position: -10rem 0;
  background-repeat: no-repeat;
}

.font-grade-1 {
  color: #8df901
}
.item-icon-popup1 {
  background-image: radial-gradient(ellipse at bottom,
  #8df90140 0,
  transparent 70%);
  background-position: -10rem 0;
  background-repeat: no-repeat;
}

.font-grade-2 {
  color: #00b0fa
}
.item-icon-popup2 {
  background-image: radial-gradient(ellipse at bottom,
  #00b0fa40 0,
  transparent 70%);
  background-position: -10rem 0;
  background-repeat: no-repeat;
}

.font-grade-3 {
  color: #ba00f9
}
.item-icon-popup3 {
  background-image: radial-gradient(ellipse at bottom,
  #ba00f940 0,
  transparent 70%);
  background-position: -10rem 0;
  background-repeat: no-repeat;
}

.font-grade-4 {
   color: #f99200
 }
.item-icon-popup4 {
  background-image: radial-gradient(ellipse at bottom,
  #f9920040 0,
  transparent 70%);
  background-position: -10rem 0;
  background-repeat: no-repeat;
}

.font-grade-5 {
  color: #fa5d00
}
.item-icon-popup5 {
   background-image: radial-gradient(ellipse at bottom,
   #fa5d0040 0,
   transparent 70%);
   background-position: -10rem 0;
   background-repeat: no-repeat;
 }

.item-card {
  background-size: 95%;
  background-position: 0.25rem 0.5rem;
  background-repeat: no-repeat;
}

.item-crew {
  background-size: cover;
  background-repeat: no-repeat;
}

.fit-content {
  width: fit-content;
  margin: auto;
}

.item-icon {
  background-size: 100%;
  background-repeat: no-repeat;
}

.item-cost {
  position: absolute;
  display: inline-block;
  width: auto;
  height: auto;
  bottom: 0;
  right: 0;
  text-shadow: -2px 0 black, 0 2px black, 2px 0 black, 0 -2px black;
  font-size: large;
  font-weight: bold;
}

.item-count {
  position: absolute;
  display: inline-block;
  width: auto;
  height: auto;
  bottom: 0;
  left: 5px;
  text-shadow: -2px 0 black, 0 2px black, 2px 0 black, 0 -2px black;
  font-size: large;
  font-weight: bold;
}

.item-sub-icon {
  position: absolute;
  display: inline-block;
  bottom: 0;
  right: 0;
}

.item-sub-icon-popup {
  position: relative;
  display: inline-block;
  width: 28px;
  height: 28px;
  bottom: 28px;
  right: -36px;
}

.item-type {
  font-weight: bold;
}

</style>