<template lang="pug">
  #app
    link(rel="stylesheet" href="https://unpkg.com/leaflet@1.6.0/dist/leaflet.css")
    input(type="checkbox" id="menuSwitcher")
    .sidemenu
      .top
        .text
          span.today {{ today.year }}/{{ today.month }}/{{ today.day }}{{ daylist[today.whatday] }}口罩在哪兒？
        .select-group
          select(v-model="select.city" @change="select.area=''")
            option -請選擇-
            option(v-for="c in cityData") {{ c.CityName }}
          select(v-model="select.area" @change="updateMarker")
            option -請選擇-
            option(v-for="a in cityData.find((city) => city.CityName === select.city).AreaList") {{ a.AreaName}}
      .infos
        template(v-for="item in data")
          .card(v-if="item.properties.county === select.city && item.properties.town === select.area",
                @click="panTo(item)")
            .title
              span.h4 {{ item.properties.name }}
              i.fa-star(:class="{fas: item.properties.custom_note=='isStar', far: item.properties.custom_note!='isStar'}" @click="item.properties.custom_note='isStar'")
            h6
              a(:href="`https://www.google.com.tw/maps/place/${item.properties.address}`"
                target="_blank") {{ item.properties.address }}
            h6 {{ item.properties.phone }}
            .mask
              span.adult(:class="{have: item.properties.mask_adult!=0, none: item.properties.mask_adult==0}") 成人口罩
                strong {{ item.properties.mask_adult}}
              span.child(:class="{have: item.properties.mask_child!=0, none: item.properties.mask_child==0}") 兒童口罩
                strong {{ item.properties.mask_child }}
            h6 {{ item.properties.note }}
            .time
              small 更新時間 {{ item.properties.updated }}
      label(for="menuSwitcher" @click="menuOpen=!menuOpen")
        i.fas.fa-angle-left(:style="{'display': menuOpen? 'block':'none'}")
        i.fas.fa-angle-right(:style="{'display': menuOpen? 'none':'block'}")
    #map
</template>

<script>
import L from 'leaflet';
import cityData from './assets/cityName.json';

let map = {};

const iconsConfig = {
   iconSize: [50, 50],
  iconAnchor: [25, 25],
  popupAnchor: [0, -25],
  shadowSize: [50, 50]
};
const icons = {
  default: new L.Icon({
    iconUrl: 'https://cdn2.iconfinder.com/data/icons/seo-flat-6/128/15_Place_Optimization-512.png',
    ...iconsConfig
  }),
};

const Today = new Date();

export default {
  name: 'App',
  data: () => ({
    data: {},
    today: {
      year: Today.getFullYear(),
      month: Today.getMonth() + 1,
      day: Today.getDate(),
      whatday: Today.getDay(),
    },
    daylist: ['（日）', '（一）', '（二）', ' （三）', '（四）', '（五）', '（六）'],
    cityData,
    select: {
      city: '臺北市',
      area: '中正區'
    },
    menuOpen: true
  }),
  methods: {
    updateMarker() {
      map.eachLayer((layer) => {
        if(layer instanceof L.Marker) {
          map.removeLayer(layer);
        }
      });
      const pharmacies = this.data.filter((pharmacy) => {
        if(!this.select.area) { //未選擇區
          return pharmacy.properties.county === this.select.city
        }
        return pharmacy.properties.county === this.select.city && pharmacy.properties.town === this.select.area 
      });
      pharmacies.forEach((pharmacy) => {
        L.marker([
          pharmacy.geometry.coordinates[1],
          pharmacy.geometry.coordinates[0]],
          {icon: icons.default})
          .addTo(map)
          .bindPopup(`<div class="popup">
          <div class="main">
          <h4>${pharmacy.properties.name}</h4>
          <h6><a href="https://www.google.com.tw/maps/place/${pharmacy.properties.address}" target="_blank">${pharmacy.properties.address}</a></h6>
          <h6>${pharmacy.properties.phone}</h6>
          </div>
          <div class="content">
          <div class="mask">
          <h5 class="adult">成人口罩<strong>${pharmacy.properties.mask_adult}</strong></h5>
          <h5 class="child">兒童口罩<strong>${pharmacy.properties.mask_child}</strong></h5>
          </div>
          <div class="time">
          <small>更新時間${pharmacy.properties.updated}</small>
          </div>
          </div>
          </div>`);
      });
      map.panTo([pharmacies[0].geometry.coordinates[1], pharmacies[0].geometry.coordinates[0]]);
    },
    panTo(pharmacy) {
      map.panTo([pharmacy.geometry.coordinates[1], pharmacy.geometry.coordinates[0]]);
      L.marker([
          pharmacy.geometry.coordinates[1],
          pharmacy.geometry.coordinates[0]],
          {icon: icons.default})
          .addTo(map)
          .bindPopup(`<div class="popup">
          <div class="main">
          <h4>${pharmacy.properties.name}</h4>
          <h6><a href="https://www.google.com.tw/maps/place/${pharmacy.properties.address}" target="_blank">${pharmacy.properties.address}</a></h6>
          <h6>${pharmacy.properties.phone}</h6>
          </div>
          <div class="content">
          <div class="mask">
          <h5 class="adult">成人口罩<strong>${pharmacy.properties.mask_adult}</strong></h5>
          <h5 class="child">兒童口罩<strong>${pharmacy.properties.mask_child}</strong></h5>
          </div>
          <div class="time">
          <small>更新時間${pharmacy.properties.updated}</small>
          </div>
          </div>
          </div>`)
          .openPopup();
    }
  },
  mounted() {
    map = L.map('map', {
      // center: [25.029532, 121.518592],
      zoom: 16
    });
    L.tileLayer('https://cartodb-basemaps-{s}.global.ssl.fastly.net/light_all/{z}/{x}/{y}.png', {
        attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
    }).addTo(map);
    this.$http.get('https://raw.githubusercontent.com/kiang/pharmacies/master/json/points.json')
    .then((res) => {
      this.data = res.data.features;
      this.updateMarker();
    });
  }
}
</script>

<style lang="sass">
@import url('https://fonts.googleapis.com/css2?family=Noto+Sans+TC:wght@300;400;500;700;900&display=swap')
@import url('https://cdnjs.cloudflare.com/ajax/libs/font-awesome/5.12.1/css/all.min.css')

$blue_dark: #4B6DE4
$blue_light: #63D2FF
$green_dark: #BED8D4
$green_light: #78D5D7
$grey_dark: #4F4F4F
$grey_light: #BEBEBE

@mixin flex-center
  display: flex
  justify-content: center
  align-items: center

*
  font-family: 'Noto Sans TC', sans-serif
html, body, #app
  width: 100%
  height: 100vh
  margin: 0
  padding: 0
#map
  width: 100%
  height: 100%
#menuSwitcher
  position: absolute
  display: none
  &:checked + .sidemenu
    transform: translateX(-300px)
.sidemenu
  width: 300px
  height: 100vh
  position: absolute
  background-color: #fff
  z-index: 1001
  top: 0
  left: 0
  transition: 0.5s
  .top
    height: 120px
    background: linear-gradient(120deg, $blue_dark, $green_light)
    padding: 15px 0px
    box-sizing: border-box
    .text
      text-align: center
      span.today
        font-size: 20px
        color: $grey_dark
    .select-group
      +flex-center
      select
        width: 120px
        margin: 10px
        padding: 5px
        border: none
        border-radius: 15px
        font-size: 16px
        color: $blue_dark
        &:focus
          outline: none
  .infos
    height: calc(100vh - 120px)
    overflow-y: scroll
    &::-webkit-scrollbar
      display: none
    &:hover::-webkit-scrollbar
      display: block
    .card
      width: 295px
      box-sizing: border-box
      border-bottom: 1px solid $grey_light
      cursor: pointer
      padding: 15px 15px
      &::last-child
        border-bottom: none
      .title
        display: flex
        justify-content: space-between
        align-items: center
        .h4
          font-size: 20px
          font-weight: 700
          padding: 0
          margin: 0
          color: $grey_dark
        i
          font-size: 20px
          color: $blue_dark
          cursor: pointer
      h6
        font-size: 14px
        font-weight: 400
        padding: 0
        margin: 5px 0
        color: $grey_dark
        a:link, a:visited
          color: $blue_dark
        a:hover, a:active
          color: $blue_light
      .mask
        font-size: 16px
        font-weight: 700
        box-shadow: 0px 0px 20px rgba($blue_dark, .3)
        margin: 20px 0
        padding: 10px 0
        border-radius: 15px
        display: flex
        justify-content: center
        align-items: center
        span
          padding: 0 5px
          &.adult.have
            color: $blue_dark
          &.child.have
            color: $green_light
          &.none
            color: $grey_light
        strong
          font-size: 20px
          font-weight: 900
          margin-left: 5px
      .time
        text-align: right
        small
          font-size: 12px
          color: $grey_light
  label
    +flex-center
    width: 25px
    height: 50px
    background-color: #fff
    color: #000
    position: absolute
    right: -25px
    top: 120px
    font-size: 24px
    border-radius: 0 10px 10px 0
    cursor: pointer
    i
      color: $grey_dark

.popup
  width: 280px
  height: 150px
  display: flex
  flex-direction: column
  justify-content: space-between
  h4
    font-size: 20px
    font-weight: 700
    padding: 0
    margin: 0
    color: $grey_dark
  h5
    font-size: 16px
    font-weight: 700
    padding: 0
    margin: 3px 0
    &.adult
      color: $blue_dark
    &.child
      color: $green_light
    // &.none
    //   color: $grey_light
    strong
      font-size: 20px
      font-weight: 900
      margin-left: 5px
  h6
    font-size: 14px
    font-weight: 400
    padding: 0
    margin: 5px 0
    color: $grey_dark
    a:link, a:visited
      color: $blue_dark
    a:hover, a:active
      color: $blue_light
  small
    font-size: 11px
    display: block
    margin: 3px 0
    color: $grey_light
  .content
    display: flex
    justify-content: space-between
    align-items: flex-end

::-webkit-scrollbar
  width: 5px
::-webkit-scrollbar-track
  border-radius: 10px
::-webkit-scrollbar-thumb
  background: $grey_light
  border-radius: 10px
::-webkit-scrollbar-thumb:hover
  background: $grey_dark
</style>
