<template>
  <div id="app">
    <Header @item-selected="setSelectedItem" />
    <WarehouseMap :selectedItem="selectedItem" />
    <Footer />
  </div>
</template>

<script>
import Header from './components/VueHeader.vue';
import Footer from './components/VueFooter.vue';
import WarehouseMap from './components/WarehouseMap.vue';

export default {
  components: {
    Header,
    Footer,
    WarehouseMap,
  },
  data() {
    return {
      selectedItem: null,
    };
  },
  methods: {
    setSelectedItem(item) {
      this.selectedItem = item;
    },
  },
};
</script>