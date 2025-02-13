<template>
  <div id="app">
    <Header @item-selected="setSelectedItem" />
    <WarehouseMap :selectedItem="selectedItem" @log-complete="updateLog" />
    <Footer :logs="logs" />
  </div>
</template>

<script>
import Header from "./components/VueHeader.vue";
import Footer from "./components/VueFooter.vue";
import WarehouseMap from "./components/WarehouseMap.vue";

export default {
  components: {
    Header,
    Footer,
    WarehouseMap,
  },
  data() {
    return {
      selectedItem: null,
      logs: [],
    };
  },
  methods: {
    setSelectedItem(item) {
      this.selectedItem = item;
    },
    updateLog(logEntry) {
      this.logs.unshift(logEntry);
      if (this.logs.length > 3) {
        this.logs.pop();
      }
    },
  },
};
</script>