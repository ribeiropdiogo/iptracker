<template>
  <div class="flex flex-col h-screen max-h-screen">
    <!-- Search / Results -->
    <div class="z-20 flex justify-center relative bg-hero-pattern bg-cover px-4 pt-8 pb-32">
      <!-- Search Input --->
      <div class="w-full max-w-screen-sm">
        <h1 class="text-white text-center text-3xl pb-4">IP Address Tracker</h1>
        <div class="flex">
          <input 
            v-model="queryIp"
            class="flex-1 py-3 px-2 rounded-tl-md rounded-bl-md focus:outline-none" 
            type="text"
            placeholder="Search for any IP address or leave empty to get your own information"
          />
          <i
            @click="getIpInfo"
            class="cursor-pointer bg-black text-white px-4 rounded-tr-md rounded-br-md flex items-center fas fa-chevron-right"
          >
          </i>
        </div>
      </div>
      <!-- IP Info -->
      <IPInfo v-if="ipInfo" v-bind:ipInfo="ipInfo"/>
    </div>

    <!-- Map -->
    <div id="map" class="h-full z-10"></div>
  </div>
</template>

<script>
// @ is an alias to /src
import IPInfo from '@/components/IPInfo.vue';
import leaflet from "leaflet";
import { onMounted, ref } from 'vue';
import axios from "axios";

export default {
  name: 'HomeView',
  components: {
    IPInfo
  },
  setup() {
    let map;
    const queryIp = ref("");
    const ipInfo = ref(null);

    onMounted(() => {
      map = leaflet.map('map').setView([46.204391, 6.143158], 5);

      leaflet.tileLayer('https://api.mapbox.com/styles/v1/mapbox/streets-v11/tiles/{z}/{x}/{y}?access_token=pk.eyJ1IjoicmliZWlyb3BkaW9nbyIsImEiOiJjbGZpNjVoYXY0ZnppM3NudGpkNmxjcHVoIn0.6yKtJcbJid5Uj4qfAftddg', {
        maxZoom: 18,
        attribution: '&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>',
        id: 'mapbox/streets-v11',
        tileSize: 512,
        zoomOffset: -1,
        accessToken: "pk.eyJ1IjoicmliZWlyb3BkaW9nbyIsImEiOiJjbGZpNjVoYXY0ZnppM3NudGpkNmxjcHVoIn0.6yKtJcbJid5Uj4qfAftddg"
      }).addTo(map);
    });

    const getIpInfo = async () => {
      try {
        const data = await axios.get(`https://geo.ipify.org/api/v2/country,city?apiKey=at_1hc6giAgqxAmoRVtW7on3tOC0eNO2&ipAddress=${queryIp.value}`);

        const result = data.data;
        ipInfo.value = {
          address: result.ip,
          location: result.location.region,
          timezone: result.location.timezone,
          isp: result.isp,
          lat: result.location.lat,
          lon: result.location.lng,

        }

        console.log(result)

        leaflet.marker([ipInfo.value.lat, ipInfo.value.lon]).addTo(map);
        map.setView([ipInfo.value.lat, ipInfo.value.lon], 13);
      }

      catch(err) {
        alert(err.message);
      }
    }

    return { queryIp, ipInfo, getIpInfo };
  },
}
</script>
