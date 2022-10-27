<script>
import Clock from "../components/Clock.vue";
import { RouterLink } from "vue-router";
export default {
  name: "HomeView",
  components: {
    Clock,
    RouterLink
  },
  data() {
    return {
      stations: null,
      search: null,
      filteredStations: [],
    };
  },
  mounted() {
    // fetch data from the API every 0 seconds
    this.getStations();
  },
  methods: {
    getStations() {
      var apiUrl = "https://api.tfl.gov.uk/StopPoint/Mode/tube";
      const res = fetch(apiUrl)
        .then((response) => {
          return response.json();
        })
        .then((data) => {
          // if the stoppointNaptan doesn't start with 940GZZLU, remove it
          // this is a hack to remove the non-tube stations
          data = data.stopPoints.filter((station) => {
            return station.naptanId.startsWith("940GZZLU");
          });
          // sort stations alphabetically
          
          // Add a property that is the last three digits of the naptanId
          // this is used to create the URL
          data.forEach((station) => {
            station.shortId = station.naptanId.slice(-3);
            station.goodName = station.commonName.replace(
              " Underground Station",
              ""
            );
          });
          data.sort((a, b) => {
            if (a.commonName < b.commonName) {
              return -1;
            }
            if (a.commonName > b.commonName) {
              return 1;
            }
            return 0;
          });
          this.stations = data;
        })
      },
      filterStationsBySearch() {
        if (this.search) {
          this.filteredStations = this.stations.filter((station) => {
            return station.commonName
              .toLowerCase()
              .includes(this.search.toLowerCase());
          });
        } else {
          this.filteredStations = null;
          return this.stations;
        }
      },
      
    },
    watch: {
      search: function (val) {
        this.filteredStations = this.stations.filter((station) => {
          return station.commonName.toLowerCase().includes(val.toLowerCase());
        });
      },
    },
  }
</script>

<template>
  <main>
    <Clock />
    <div class="flex w-full px-2 flex-wrap">
      <div class="w-full justify-center">
      <input class=" max-w-sm w-small bg-gray-800 p-2 mb-4 w-96 mx-auto flex text-white rounded-md border-none" type="text" placeholder="Search for Station" v-model="search" />

      </div>
      <ul v-if="!filteredStations.length" class="flex w-full flex-row flex-wrap">
        <li
          class="w-full sm:w-1/2 md:w-1/3 2xl:w-1/4"
          v-for="(station, indx) in stations"
          v-bind:key="indx"
        >
          <RouterLink :to="'/' + station.shortId">
            <div class="py-1">
              
              <div class="text-lg text-white font-bold">
                {{ station.goodName }}
              </div>
            </div>
          </RouterLink>
        </li>
      </ul>
      <ul v-if="filteredStations.length" class="flex w-full flex-row flex-wrap">
        <li
          class="w-full sm:w-1/2 md:w-1/3 2xl:w-1/4"
          v-for="(station, indx) in filteredStations"
          v-bind:key="indx"
        >
          <RouterLink :to="'/' + station.shortId">
            <div class="py-1">
              
              <div class="text-lg text-white font-bold">
                {{ station.goodName }}
              </div>
            </div>
          </RouterLink>
        </li>
      </ul>
    </div>
    
  </main>
</template>
