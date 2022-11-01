<template>
  <div
    class="rounded-sm bg-gray-800 p-2 text-amber-500 m-2 uppercase flex flex-row flex-wrap border-l-4"
    :class="'border-' + train.lineId"
  >
    <div class="flex-1">{{ train.towards }}</div>
    <div class="flex-0 w-20 text-left flex flex-row">

      <div class="w-10 text-right pr-2">{{ (train.timeToStation / 60) | floor }}</div>
      <div>
       min<span
        v-if="(train.timeToStation / 60) | (floor > 1)"
        >s</span
      >
      </div>
    </div>
    <div class="w-full text-amber-400 text-xs">{{ train.currentLocation }}</div>
  </div>
</template>
<script>
export default {
  props: {
    train: {
      type: Object,
      required: true,
    },
  },
  data() {
    return {
      // an array of the correct colours for each underground line
    };
  },
};
</script>
