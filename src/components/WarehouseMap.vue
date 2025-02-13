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
    <!-- 선택한 물품의 시작 위치에 아이콘을 보이도록 설정 -->
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
        { name: "서울", position: { top: "23%", left: "50%" } },
        { name: "대전", position: { top: "45%", left: "48%" } },
        { name: "광주", position: { top: "75%", left: "39%" } },
        { name: "부산", position: { top: "82%", left: "70%" } },
        { name: "대구", position: { top: "70%", left: "65%" } },
      ],
      itemPaths: {
        물품1: ["서울", "대전", "광주"],
        물품2: ["서울", "부산", "대구", "대전"],
        물품3: ["서울", "대구", "광주", "부산", "대전"],
        물품4: ["부산", "대전", "서울", "광주", "대구"],
        물품5: ["광주", "서울", "부산", "대구"],
      },
      currentItemIndex: 0,
      moveProgress: 0,
      interval: null,
      // 각 물품에 해당하는 아이콘을 정의합니다.
      itemIcons: {
        물품1: require("@/assets/item1.png"),
        물품2: require("@/assets/item2.png"),
        물품3: require("@/assets/item3.png"),
        물품4: require("@/assets/item4.png"),
        물품5: require("@/assets/item5.png"),
      },
      resetting: false,  // 변환 중 상태를 추적
    };
  },
  watch: {
    selectedItem(newItem, oldItem) {
      if (newItem !== oldItem) {
        this.resetAnimation();
        setTimeout(() => {
          this.animateItem();
        }, 100);  // 짧은 지연 후 애니메이션 시작
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
        this.resetting = false;  // 리셋 후 아이콘을 표시
      }, 50);  // 리셋 후 짧은 지연으로 새로운 아이콘 표시
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

      // 최종 위치에 도달했으면 마지막 위치에 고정
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

      // 1초 대기 후 애니메이션 시작
      setTimeout(() => {
        const moveItem = () => {
          this.moveProgress += 0.2; // 이동 진행도 증가

          if (this.moveProgress >= 1) {
            this.moveProgress = 0;
            this.currentItemIndex++;
            if (this.currentItemIndex >= path.length - 1) {
              clearInterval(this.interval);
              this.interval = null; // 모든 경로 완료 후 interval 해제
              return;
            }
          }
        };

        // 물품의 이동 시작
        this.interval = setInterval(moveItem, 200); // 0.2초마다 이동하여 1초에 걸쳐 각 스텝 완료
      }, 1000); // 최초 위치에서 1초 대기
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