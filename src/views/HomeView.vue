<template>
  <div class="flex flex-col h-screen max-h-screen">
    <div
      class="z-20 flex justify-center relative bg-hero-pattern bg-cover px-4 pt-8 pb-32"
    >
      <div class="w-full max-w-screen-sm">
        <h1 class="text-white text-center text-3xl pb-4">IP Address Tracker</h1>

        <div class="flex">
          <input
            type="text"
            class="flex-1 py-3 px-2 rounded-tl-md rounded-bl-md focus:outline-none"
            placeholder="Search..."
            v-model="queryIp"
          />
          <i
            @click="getIpInfo"
            class="cursor-pointer flex items-center bg-black text-white px-4 rounded-tr-md rounded-br-md fa-solid fa-chevron-right"
          ></i>
        </div>
      </div>

      <IPinfo v-if="ipInfo" :ipInfo="ipInfo" />
    </div>

    <div id="map" class="h-full z-10"></div>
  </div>
</template>

<script>
// @ is an alias to /src
import IPinfo from "@/components/IPinfo.vue";
import leaflet from "leaflet";
import { onMounted, ref } from "@vue/runtime-core";
import axios from "axios";

export default {
  name: "HomeView",
  components: {
    IPinfo,
  },
  setup() {
    let mymap;

    const queryIp = ref("");
    const ipInfo = ref(null);

    onMounted(() => {
      mymap = leaflet.map("map").setView([51.505, -0.09], 13);

      leaflet
        .tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
          maxZoom: 19,
          attribution: "© OpenStreetMap",
        })
        .addTo(mymap);
    });

    let marker;
    const getIpInfo = async () => {
      try {
        const data = await axios.get(
          `https://geo.ipify.org/api/v2/country,city?apiKey=at_Bmu1jHnFBBRpXE4YUbNRnVWK6tmRH&ipAddress=${queryIp.value}`
        );
        const result = data.data;
        ipInfo.value = {
          address: result.ip,
          state: result.location.region,
          timezone: result.location.timezone,
          isp: result.isp,
          lat: result.location.lat,
          lng: result.location.lng,
        };
        if (marker) {
          mymap.removeLayer(marker);
        }
        marker = leaflet
          .marker([ipInfo.value.lat, ipInfo.value.lng])
          .addTo(mymap);
        mymap.setView([ipInfo.value.lat, ipInfo.value.lng], 13);
      } catch (error) {
        alert(error.message);
      }
    };

    return { queryIp, ipInfo, getIpInfo };
  },
};
</script>
