<template>
  <div id="map-container">
    <img src="/map.svg" alt="Map" id="map-background" />
    <div
      v-for="warehouse in warehouses"
      :key="warehouse.name"
      :class="['warehouse', { highlighted: warehouse.name === currentLocation, visited: isVisited(warehouse.name) }]"
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
import rfidData from "@/data/rfid-data.json"; // JSON 파일을 가져오기

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
      itemPaths: {
        물품1: ["대전", "광주"],    // json의 최초 위치를 제외한 경로
        물품2: ["부산", "대구", "대전"],
        물품3: ["대구", "광주", "부산", "대전"],
        물품4: ["대전", "서울", "광주", "대구"],
        물품5: ["서울", "부산", "대구"],
      },
      rfidData: rfidData,
      currentLocation: null,
      startLocation: null,
      visitedLocations: [],
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
        this.updateStartLocation();
        this.animateItem();
      }
    },
  },
  methods: {
    updateStartLocation() {
      const latestEntry = this.rfidData
        .filter((entry) => entry.id === this.selectedItem)
        .sort((a, b) => new Date(b.time) - new Date(a.time))[0];

      if (latestEntry) {
        this.startLocation = latestEntry.location;
      }
    },
    animateItem() {
      if (!this.selectedItem) return;

      const path = [this.startLocation, ...this.itemPaths[this.selectedItem]];
      this.currentLocation = path[0];
      this.visitedLocations = [{ location: this.currentLocation, time: new Date().toLocaleString() }];
      let stepIndex = 1;

      // 1초 대기 후 애니메이션 시작
      setTimeout(() => {
        this.interval = setInterval(() => {
          if (stepIndex < path.length) {
            this.currentLocation = path[stepIndex];
            this.visitedLocations.push({ location: this.currentLocation, time: new Date().toLocaleString() });
            stepIndex++;
          } else {
            clearInterval(this.interval);
            this.$emit("log-complete", {
              itemName: this.selectedItem,
              path: this.visitedLocations,
            });
          }
        }, 1000);
      }, 1000);
    },
    resetAnimation() {
      this.currentLocation = null;
      this.visitedLocations = [];
      this.resetting = true;
      setTimeout(() => {
        this.resetting = false;
      }, 50);
      clearInterval(this.interval);
    },
    isVisited(location) {
      return this.visitedLocations.some(visit => visit.location === location);
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

.warehouse.visited {
  background-color: lightblue;
}

.warehouse.highlighted {
  background-color: yellow;
}

.item-icon {
  width: 25px;
  height: 25px;
  position: absolute;
}
</style>