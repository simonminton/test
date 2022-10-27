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
  <main>
    <DisplayBoard />
    <Clock />
  </main>
</template>