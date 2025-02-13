<template>
  <div id="map-container">
    <img src="/map.svg" alt="Map" id="map-background" />
    <div
      v-for="warehouse in warehouses"
      :key="warehouse.name"
      :class="['warehouse', { highlighted: isHighlighted(warehouse.name) }]"
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
export default {
  props: ["selectedItem"],
  data() {
    return {
      warehouses: [
        // 각 지역의 위치를 적절히 수정하여 반영
        { name: '서울', position: { top: '20%', left: '32%' } },
        { name: '대전', position: { top: '42%', left: '40%' } },
        { name: '광주', position: { top: '63%', left: '29%' } },
        { name: '부산', position: { top: '63%', left: '77%' } },
        { name: '대구', position: { top: '51%', left: '67%' } },
        { name: '울산', position: { top: '55%', left: '81%' } },
      ],
      itemPaths: {
        '물품1': ['서울', '대전', '광주', '울산'],
        '물품2': ['서울', '부산', '대구', '대전', '울산'],
        '물품3': ['서울', '대구', '광주', '부산', '대전', '울산'],
        '물품4': ['부산', '대전', '서울', '광주', '대구','울산'],
        '물품5': ['광주', '서울', '부산', '대구','울산'],
      },
      currentItemIndex: 0,
      moveProgress: 0,
      interval: null,
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
    selectedItem(newItem, oldItem) {
      if (newItem !== oldItem) {
        this.resetAnimation();
        setTimeout(() => {
          this.animateItem();
        }, 100);
      }
    },
  },
  methods: {
    isHighlighted(name) {
      return (
        this.selectedItem &&
        this.itemPaths[this.selectedItem].includes(name)
      );
    },
    resetAnimation() {
      clearInterval(this.interval);
      this.interval = null;
      this.currentItemIndex = 0;
      this.moveProgress = 0;
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
      const path = this.itemPaths[this.selectedItem];
      const originWarehouse = this.warehouses.find(
        (w) => w.name === path[this.currentItemIndex]
      );
      const destinationWarehouse = this.warehouses.find(
        (w) => w.name === path[this.currentItemIndex + 1]
      );

      const finalTopPercent = parseFloat(originWarehouse.position.top);
      const finalLeftPercent = parseFloat(originWarehouse.position.left);

      const currentTopPercent = destinationWarehouse
        ? parseFloat(originWarehouse.position.top) +
          (parseFloat(destinationWarehouse.position.top) -
            parseFloat(originWarehouse.position.top)) *
            this.moveProgress
        : finalTopPercent;

      const currentLeftPercent = destinationWarehouse
        ? parseFloat(originWarehouse.position.left) +
          (parseFloat(destinationWarehouse.position.left) -
            parseFloat(originWarehouse.position.left)) *
            this.moveProgress
        : finalLeftPercent;

      return {
        top: `${currentTopPercent}%`,
        left: `${currentLeftPercent}%`,
        transform: "translate(-50%, -50%)",
        transition: "top 0.2s linear, left 0.2s linear",
      };
    },
    animateItem() {
      if (!this.selectedItem || this.interval) return;

      const path = this.itemPaths[this.selectedItem];
      this.currentItemIndex = 0;
      this.moveProgress = 0;

      setTimeout(() => {
        const moveItem = () => {
          this.moveProgress += 0.2;

          if (this.moveProgress >= 1) {
            this.moveProgress = 0;
            this.currentItemIndex++;
            if (this.currentItemIndex >= path.length - 1) {
              clearInterval(this.interval);
              this.interval = null;
              const logEntry = {
                itemName: this.selectedItem,
                location: path[this.currentItemIndex],
                time: new Date().toLocaleString(),
              };
              this.$emit("log-complete", logEntry);
              return;
            }
          }
        };

        this.interval = setInterval(moveItem, 200);
      }, 1000);
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