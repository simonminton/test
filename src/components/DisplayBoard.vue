<template>
  <div class="w-full flex flex-row flex-wrap-reverse md:flex-wrap">
    <div class="w-full md:w-1/4 bg-gray-800 text-white p-4">
      <RouterLink to="/"> Home </RouterLink>
      <RouterLink class="py-1" :to="'/' + $route.params.station">
        &rarr; {{ this.stationName }}
      </RouterLink>
      <RouterLink
        v-if="$route.params.line"
        class="py-1 uppercase"
        :to="'/' + $route.params.station + '/' + $route.params.line"
      >
        &rarr; {{ $route.params.line }}
      </RouterLink>
      <RouterLink
        v-if="$route.params.direction"
        class="py-1"
        :to="
          '/' +
          $route.params.station +
          '/' +
          $route.params.line +
          '/' +
          $route.params.direction
        "
      >
        &rarr; {{ $route.params.direction }}
      </RouterLink>
      <div
        v-if="lines"
        class="text-white font-semibold border-b border-gray-600 my-2 pb-2 mt-4"
      >
        Lines
      </div>
      <div class="flex flex-row flex-wrap">
      <div
        v-for="(line, lindx) in lines"
        v-bind:key="lindx"
        class="text-white w-full pr-2 pb-2 mt-4 text-sm font-semibold uppercase"
      >
        <RouterLink
          class="px-2 py-1"
          :to="'/' + $route.params.station + '/' + line.lineName"
          :class="$route.params.line == line.lineName ? 'border-white box-border border-2 bg-' + line.lineName : 'border-' + line.lineName + ' box-border border-2 bg-' + line.lineName">
          {{ line.lineName }}
      </RouterLink>
      </div>
      </div>
      <div
        v-if="$route.params.line"
        class="text-white font-semibold border-b border-gray-600 pb-2 mt-4"
      >
        Options
      </div>
      <div v-if="$route.params.line" class="text-white pb-2 mt-4 flex flex-row">
        <RouterLink
          class="w-1/2 block text-center"
          :to="
            '/' + $route.params.station + '/' + $route.params.line + '/inbound'
          "
        >
          Inbound
      </RouterLink>
        <RouterLink
          class="w-1/2 block text-center"
          :to="
            '/' + $route.params.station + '/' + $route.params.line + '/outbound'
          "
        >
          Outbound
    </RouterLink>
      </div>
    </div>
    <ul
      v-if="trainData"
      class="w-full md:w-3/4 rounded-md px-4 md:px-8 max-w-xl mx-auto dotmatrix"
    >
      <li v-for="(train, indx) in trainData" v-bind:key="indx">
        <SingleTrain :train="train" />
      </li>
    </ul>
    <div v-else>
      <div class="text-white text-center">Loading data...</div>
    </div>
  </div>
</template>

<script>
import SingleTrain from "../components/SingleTrain.vue";
import { RouterLink } from "vue-router";

export default {
  components: {
    SingleTrain,
    RouterLink
  },
  props: {
    msg: {
      type: String,
      required: true,
    },
  },
  data() {
    return {
      trainData: null,
      lines: null,
      stationName: null,
      trainTime: null
    };
  },
  mounted() {
    // fetch data from the API every 0 seconds
    this.getTrainData();
    this.getStationLines();
  },
  watch:{
    $route: {
     handler: function(value) {
        this.trainTime = 0;
        this.trainData = null;
        this.getTrainData();
    },
      deep: true,
      immediate: true,
  }
},
  methods: {
    getStationLines() {
      // str to uppr
      var station = this.$route.params.station.toUpperCase();
      var apiUrl =
        "https://api.tfl.gov.uk/StopPoint/ServiceTypes?id=940GZZLU" +
        station +
        "&modes=tube";
      // Fetch train data from the API
      const res = fetch(apiUrl)
        .then((response) => {
          return response.json();
        })
        .then((data) => {
          // Sort data by due time

          this.lines = data;
        });
    },
    getTrainData() {
      var station = this.$route.params.station.toUpperCase();
      var apiUrl =
        "https://api.tfl.gov.uk/StopPoint/940GZZLU" + station + "/arrivals";
      if (this.$route.params.line) {
        apiUrl =
          "https://api.tfl.gov.uk/Line/" +
          this.$route.params.line +
          "/Arrivals/940GZZLU" +
          station;
      }
      if (this.$route.params.direction) {
        apiUrl =
          "https://api.tfl.gov.uk/Line/" +
          this.$route.params.line +
          "/Arrivals/940GZZLU" +
          station +
          "?direction=" +
          this.$route.params.direction;
      }
      if (this.$route.params.destination) {
        apiUrl =
          "https://api.tfl.gov.uk/Line/" +
          this.$route.params.line +
          "/Arrivals/940GZZLU" +
          station +
          "?direction=" +
          this.$route.params.direction +
          "&destinationStationId=940GZZLU" +
          this.$route.params.destination;
      }
      var now = new Date().getTime();
      // if time was more than 5 seconds ago, fetch new data
      if (this.trainData == null || now - this.trainTime > 5000) {
        this.trainTime = new Date().getTime();
      // Fetch train data from the API
      const res = fetch(apiUrl)
        .then((response) => {
          return response.json();
        })
        .then((data) => {
          // Sort data by due time
          data.sort((a, b) => (a.timeToStation > b.timeToStation ? 1 : -1));
          this.trainData = data;
          // Remove Underground Station from name if it exists
          this.stationName = data[0].stationName.replace(
            " Underground Station",
            ""
          );
          
        });
      }
      setTimeout(this.getTrainData, 1000);
    },
  },
};
</script>