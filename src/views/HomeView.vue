<template>
  <div class="w-full h-screen max-h-screen flex flex-col">
    <div
      class="
        flex
        justify-center
        relative
        z-20
        items-center
        mx-auto
        w-full
        bg-cover
        pt-8
        pb-32
        px-4
        flex-col
        bg-hero-pattern
      "
    >
      <h1 class="text-2xl text-white max-w-screen-sm">IP address tracker</h1>

      <div
        class="
          flex
          w-full
          items-center
          justify-between
          rounded-md
          max-w-screen-sm
          mt-10
        "
      >
        <input
          type="text"
          v-model="queryIP"
          name="search"
          placeholder="Search for any IP address or leave it empty to get your IP information"
          class="
            w-full
            px-4
            py-2
            focus:outline-none focus:shadow-outline
            rounded-tl-md rounded-bl-md
          "
        />

        <font-awesome-icon
          @click="searchIP"
          class="
            text-2xl text-white
            bg-black
            px-4
            py-2
            rounded-tr-md rounded-br-md
            hover:bg-gray-500 hover:transition-all
            duration-700
            ease-in-out
            hover:cursor-pointer
          "
          :icon="['fa-solid', 'fa-angle-right']"
          width="20px"
        />
      </div>
      <IPInfo v-if="IpInfo" :Info="IpInfo" @close="IpInfo = null" />
    </div>
    <div id="map" class="z-10 w-full h-full"></div>
  </div>
</template>

<script>
import IPInfo from "../components/IPInfo.vue";
import leaflet from "leaflet";
import { onMounted, ref } from "vue";
import axios from "axios";
var validate = require("ip-validator");
export default {
  components: {
    IPInfo,
  },
  setup() {
    const queryIP = ref(null);
    const IpInfo = ref(null);
    let myMap;
    onMounted(() => {
      myMap = leaflet.map("map").setView([31.4175388, 31.8144434], 12);
      leaflet
        .tileLayer(
          `https://api.mapbox.com/styles/v1/{id}/tiles/{z}/{x}/{y}?access_token=${process.env.VUE_APP_API_KEY}`,
          {
            attribution:
              'Map data &copy; <a href="https://www.openstreetmap.org/">OpenStreetMap</a> contributors, <a href="https://creativecommons.org/licenses/by-sa/2.0/">CC-BY-SA</a>, Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
            maxZoom: 18,
            id: "mapbox/streets-v11",
            tileSize: 512,
            zoomOffset: -1,
            access_token: process.env.VUE_APP_API_KEY,
          }
        )
        .addTo(myMap);
    });

    const searchIP = async () => {
      const url = `${process.env.VUE_APP_BASE_URL}`;
      const RapidAPI = `${process.env.VUE_APP_GEO_API_KEY}`;
      let ip = queryIP.value;
      if (!ip) {
        const ipTwo = await axios.get(
          "https://www.cloudflare.com/cdn-cgi/trace"
        );
        const resultTwo = ipTwo.data.split("\n")[2].split("=")[1];
        ip = resultTwo;
      }
      var valid = validate.ipv4(ip);
      console.log(valid);
      if (valid) {
        const options = {
          method: "GET",
          url: url,
          params: { ip: ip },
          headers: {
            "X-RapidAPI-Key": RapidAPI,
            "X-RapidAPI-Host": "ip-geolocation-ipwhois-io.p.rapidapi.com",
          },
        };
        try {
          const data = await axios.request(options);
          const result = data.data;
          const shownData = {
            "IP Address": result.ip,
            Location: `${result.country}, ${result.city}, ${result.region}`,
            Timezone: `${result.timezone}, ${result.timezone_name}, ${result.timezone_gmt}`,
            ISP: `${result.isp}, ${result.org}`,
          };
          IpInfo.value = shownData;
          myMap.setView([result.latitude, result.longitude], 13);
          leaflet
            .marker([result.latitude, result.longitude])
            .addTo(myMap)
            .bindPopup(
              `<b>${result.ip}</b><br>${shownData.Location}<br>lat: ${result.latitude}, <br>lng: ${result.longitude}`
            )
            .openPopup();
          queryIP.value = "";
        } catch (error) {
          IpInfo.value = null;
          alert(`${error.message}`);
          queryIP.value = "";
        }
      } else {
        alert("Please enter a valid IP address");
        queryIP.value = "";
      }
    };

    return { queryIP, IpInfo, searchIP };
  },
};
</script>

<style>
</style>
