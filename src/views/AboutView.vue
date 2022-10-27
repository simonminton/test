<script>

import Clock from "../components/Clock.vue";
import DisplayBoard from "../components/DisplayBoard.vue";
export default {
  components: {
    Clock,
    DisplayBoard,
  },
}
</script>
<template>
  <main class="flex-col-reverse flex md:flex-col">
    <DisplayBoard />
    <Clock />
  </main>
</template>