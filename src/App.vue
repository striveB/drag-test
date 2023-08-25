<script setup lang="ts">
import { reactive, computed, ref } from "vue";
import { useWindowSize } from "@vueuse/core";
import draggable from "vuedraggable";
const container = reactive({
  width: 1800,
  height: 1800,
  transform: {
    translate: { x: 0, y: 0 },
    scale: 1,
  },
});

function conMove(e: MouseEvent) {
  // console.log(e);
}

type Hosue = {
  title: string;
};
let houseList: Hosue[] = reactive([
  {
    title: "房1",
  },
  {
    title: "房2",
  },
  {
    title: "房3",
  },
]);
// 生成格子
// 格子宽度
const gridWidth = 110;
// 格子高度
const gridHeight = 90;
const grids: {
  id: string;
  ring: number;
  num: number;
  px: number;
  py: number;
  houses: Hosue[];
}[] = reactive([]);
// 根据圈数生成grid
function createdGrids(num: number) {
  for (let i = 0; i < num; i++) {
    let gridNum = i * 4 * 2 + 4;
    let old = null;
    let type = 1;
    for (let j = 0; j < gridNum; j++) {
      let x = 0;
      let y = 0;
      if (!old) {
        x = (i + 1) * gridWidth * -1 + gridWidth;
        y = (i + 1) * gridHeight * -1 + gridHeight;
      } else {
        if (type === 1) {
          x = old.px + gridWidth;
          y = old.py;
        } else if (type === 2) {
          x = old.px;
          y = old.py + gridHeight;
        } else if (type === 3) {
          x = old.px - gridWidth;
          y = old.py;
        } else if (type === 4) {
          x = old.px;
          y = old.py - gridHeight;
        }
        if (j % (1 * (i + (i + 1))) === 0) {
          type++;
        }
      }
      let grid = {
        id: i + "-" + j + 1,
        ring: i,
        num: j + 1,
        px: x,
        py: y,
        houses: [],
      };
      grids.push(grid);
      old = grid;
    }
  }
}
createdGrids(7);

const onEnd = (e) => {
  console.log(e);
};
</script>

<template>
  <div class="panel">
    <draggable
      class="houses"
      :list="houseList"
      :group="{
        name: 'house',
        pull: 'clone',
        put: false,
        sort: false,
      }"
      itemKey="title"
      @end="onEnd"
    >
      <template #item="{ element }">
        <div class="house">{{ element.title }}</div>
      </template>
    </draggable>
    <div class="box" @mousemove="conMove">
      <draggable
        v-for="grid in grids"
        :group="{
          name: 'house',
          pull: false,
          put: true,
          sort: false,
        }"
        :disabled="grid.houses.length > 0 || grid.ring == 0"
        class="place"
        :list="grid.houses"
        :style="{
          width: gridWidth + 'px',
          height: gridHeight + 'px',
          left: grid.px + 'px',
          top: grid.py + 'px',
        }"
        @end="onEnd"
        itemKey="id"
      >
        <template #item="{ element }">
          <div class="house">{{ element.title }}</div>
        </template>
      </draggable>
      <div
        class="center"
        :style="{ width: gridWidth * 2 + 'px', height: gridHeight * 2 + 'px' }"
      >
        能量核心
      </div>
    </div>
  </div>
</template>

<style lang="scss" scoped>
.house {
  flex-shrink: 0;
  width: 100px;
  height: 80px;
  border-radius: 10px;
  border: 2px solid #fff;
  display: flex;
  justify-content: center;
  align-items: center;
  position: relative;
  margin: 5px;
  cursor: pointer;
  user-select: none;
  transition: all 0.3s;
  overflow: hidden;
  &:hover {
    background: rgba(255, 255, 255, 0.1);
  }
}
.panel {
  width: 100vw;
  height: 100vh;
  // overflow: hidden;
  .houses {
    display: flex;
    margin-left: 50%;
    transform: translateX(-30%);
    color: red;
  }
  .box {
    height: 100vh;
    transform: translate(46%, 68%);
    margin-left: 500px;
    margin: auto;
    position: relative;
    .place {
      position: absolute;
      padding: 5px;
      border: 1px solid rgba(255, 181, 62, 0.1);
      flex-shrink: 0;
      box-sizing: border-box;
      user-select: none;
      display: flex;
      justify-content: center;
      align-items: center;
      color: yellow;
    }
  }
  .center {
    position: absolute;
    border: 5px solid rgba(255, 181, 62, 0.8);
    border-radius: 10px;
    font-weight: bold;
    box-sizing: border-box;
    transition: all 0.4s;
    display: flex;
    justify-content: center;
    align-items: center;
    background-color: rgba(255, 181, 62, 0.1);
    &::before {
      content: "";
      display: block;
      width: 100%;
      height: 100%;
      position: absolute;
      left: 50%;
      top: 50%;
      border-radius: 10px;
      transform: translate(-50%, -50%);
      transition: all 0.3s;
      border: 2px solid rgba(255, 181, 62, 0.3);
      background-color: rgba(255, 181, 62, 0.1);
    }
    &:hover {
      box-shadow: 0px 0px 400px 0px rgba(255, 181, 62, 1);
      background-color: rgba(255, 181, 62, 0.3);
      &::before {
        content: "";
        width: 420%;
        height: 420%;
      }
    }
  }
}
</style>
