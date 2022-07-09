<template>
  <div class="flex flex-col h-screen max-h-screen">
    <!-- Search/result -->
    <div class="z-20 flex justify-center relative bg-hero-pattern bg-cover px-4 pt-8 pb-32">
      <!-- Search Input -->
      <div class="w-full max-w-screen-sm">
        <h1 class="text-white text-center text-3xl pb-4">IP Address Tracker</h1>
        <div class="flex">
          <input v-model="queryIP" class="flex-1 py-3 px-2 rounded-tl-md rounded-bl-md focus:outline-none" type="text" placeholder="Search for any IP Address or leave empty to get your IP Info">
          <i @click="getIpInfo" class="cursor-pointer bg-black text-white px-4 py-4 rounded-tr-md rounded-br-md flex item-center fas fa-chevron-right"></i>
        </div>
      </div>
      <!-- IP Info -->
      <IPInfo v-if="ipInfo" v-bind:ipInfo="ipInfo" />
    </div>
    <!-- Map -->
    <div id="mapid" class="h-full z-10"></div>
  </div>
</template>

<script>
// @ is an alias to /src
import IPInfo from '@/components/IPInfo.vue';
import leaflet from 'leaflet';
import { onMounted, ref } from 'vue';
import axios from 'axios';

export default {
  name: 'HomeView',
  components: {
    IPInfo
  },
  setup() {
    let mymap;

    const queryIP = ref ("");
    const ipInfo = ref (null);

    onMounted(() => {
      mymap = leaflet.map("mapid").setView([-6.200000, 106.816666], 4);
        leaflet.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
            maxZoom: 19,
            attribution: '© OpenStreetMap'
        }).addTo(mymap);
    });
    
    const getIpInfo = async () => {
      try {
        const data = await axios.get(`https://geo.ipify.org/api/v2/country,city?apiKey=at_LSIMEPfQ1877tGXTPi69UdUzDPTsv&ipAddress=${queryIP.value}`);
        const result = data.data;
        // console.log(result);
        ipInfo.value = {
          address: result.ip,
          state: result.location.region,
          timezone: result.location.timezone,
          isp: result.isp,
          lat: result.location.lat,
          lng: result.location.lng
        };
        leaflet.marker([ipInfo.value.lat, ipInfo.value.lng]).addTo(mymap);
        mymap.setView([ipInfo.value.lat, ipInfo.value.lng], 9);
      } catch (err) {
        alert(err.message)
      }
    }
    return { queryIP, ipInfo, getIpInfo };
  },
}
</script>
