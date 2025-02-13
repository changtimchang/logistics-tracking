<template>
  <div id="map-container">
    <img src="/map.svg" alt="Map" id="map-background" />
    <div
      v-for="warehouse in warehouses"
      :key="warehouse.name"
      :class="['warehouse', { highlighted: warehouse.name === currentLocation }]"
      :style="getPositionStyle(warehouse)"
    >
      {{ warehouse.name }}
    </div>
    <img
      v-if="selectedItem && !resetting"
      class="item-icon"
      :style="getItemPositionStyle()"
      :src="itemIcons[selectedItem]"
      alt="Item Icon"
    />
  </div>
</template>

<script>
import axios from "axios"; // 예시로 axios 사용

export default {
  props: ["selectedItem"],
  data() {
    return {
      warehouses: [
        { name: "서울", position: { top: "23%", left: "50%" } },
        { name: "대전", position: { top: "45%", left: "48%" } },
        { name: "광주", position: { top: "75%", left: "39%" } },
        { name: "부산", position: { top: "82%", left: "70%" } },
        { name: "대구", position: { top: "70%", left: "65%" } },
      ],
      rfidData: [], // RFID 데이터를 저장
      currentLocation: null, // 현재 물품의 위치
      itemIcons: {
        물품1: require("@/assets/item1.png"),
        물품2: require("@/assets/item2.png"),
        물품3: require("@/assets/item3.png"),
        물품4: require("@/assets/item4.png"),
        물품5: require("@/assets/item5.png"),
      },
      resetting: false,
    };
  },
  watch: {
    selectedItem(newItem) {
      if (newItem) {
        this.resetAnimation();
        this.fetchRFIDData();
      }
    },
  },
  methods: {
    fetchRFIDData() {
      const apiUrl = `https://api.example.com/rfid-data`;
      
      axios.get(apiUrl)
        .then(response => {
          this.rfidData = response.data;
          this.updateCurrentLocation();
        })
        .catch(error => {
          console.error("Error fetching RFID data", error);
        });
    },
    updateCurrentLocation() {
      const latestEntry = this.rfidData
        .filter(entry => entry.id === this.selectedItem)
        .sort((a, b) => new Date(b.time) - new Date(a.time))[0];
      
      if (latestEntry) {
        this.currentLocation = latestEntry.location;
      }
    },
    resetAnimation() {
      this.currentLocation = null;
      this.resetting = true;
      setTimeout(() => {
        this.resetting = false;
      }, 50);
    },
    getPositionStyle(warehouse) {
      return {
        top: warehouse.position.top,
        left: warehouse.position.left,
        transform: "translate(-50%, -50%)",
      };
    },
    getItemPositionStyle() {
      const warehouse = this.warehouses.find(
        (w) => w.name === this.currentLocation
      );

      return {
        top: warehouse ? warehouse.position.top : "0%",
        left: warehouse ? warehouse.position.left : "0%",
        transform: "translate(-50%, -50%)",
        transition: "top 0.2s linear, left 0.2s linear",
      };
    },
  },
};
</script>

<style scoped>
#map-container {
  position: relative;
  width: 100%;
  max-width: 800px;
  margin: 0 auto;
  overflow: hidden;
  height: auto;
  border: 1px solid #ccc;
}

#map-background {
  display: block;
  width: 100%;
  height: auto;
}

.warehouse {
  position: absolute;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 60px;
  height: 60px;
  border-radius: 50%;
  background-color: lightgray;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.3);
  color: black;
  font-weight: bold;
  transition: background-color 0.3s ease;
}

.highlighted {
  background-color: yellow;
}

.item-icon {
  width: 25px;
  height: 25px;
  position: absolute;
}
</style>