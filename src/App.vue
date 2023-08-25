<script setup lang="ts">
import { reactive, computed, ref } from "vue";
import { useWindowSize } from "@vueuse/core";
const container = reactive({
  width: 1800,
  height: 1800,
  transform: {
    translate: { x: 0, y: 0 },
    scale: 1,
  },
});

let isMove = ref(false);
let clickPoint = {
  x: 0,
  y: 0,
};

function conDown(e: MouseEvent) {
  isMove.value = true;
  clickPoint.x = e.clientX - container.transform.translate.x;
  clickPoint.y = e.clientY - container.transform.translate.y;
}
function conUp() {
  isMove.value = false;
}
function conMove(e: MouseEvent) {
  if (isMove.value) {
    let x = (clickPoint.x - e.clientX) * -1;
    if (x >= 0) {
      x = 0;
    }
    let y = (clickPoint.y - e.clientY) * -1;
    if (y >= 0) {
      y = 0;
    }
    container.transform.translate.x = x;
    container.transform.translate.y = y;
  }
}

let { width: winWidth, height: winHeight } = useWindowSize();
const style = computed(() => {
  let panelPosY = winHeight.value + Math.abs(container.transform.translate.y);
  let panelPosX = winWidth.value + Math.abs(container.transform.translate.x);
  if (panelPosY >= container.height) {
    container.transform.translate.y =
      (container.height - winHeight.value - 1) * -1;
  }
  if (panelPosX >= container.width) {
    container.transform.translate.x =
      (container.width - winWidth.value - 1) * -1;
  }
  return `
  width:${container.width}px;
  height:${container.height}px;
  transform: 
    translate(${container.transform.translate.x}px, ${container.transform.translate.y}px) 
    scale(${container.transform.scale})`;
});

// 生成格子
// 格子宽度
const gridWidth = 100;
// 格子高度
const gridHeight = 100;
const grids: {
  ring: number;
  num: number;
  px: number;
  py: number;
}[] = [];
// 根据圈数生成grid 4 12
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
        ring: i,
        num: j + 1,
        px: x,
        py: y,
      };
      grids.push(grid);
      old = grid;
    }
  }
}
createdGrids(7);
</script>

<template>
  <div class="panel">
    <div class="box" @mousedown="conDown" @mouseup="conUp" @mousemove="conMove">
      <div
        class="place"
        v-for="grid in grids"
        :style="{
          width: gridWidth + 'px',
          height: gridHeight + 'px',
          left: grid.px + 'px',
          top: grid.py + 'px',
        }"
      ></div>
      <div class="center">能量核心</div>
    </div>
  </div>
</template>

<style lang="scss" scoped>
.panel {
  width: 100vw;
  height: 100vh;
  // overflow: hidden;
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
    }
  }
  .center {
    position: absolute;
    width: 200px;
    height: 200px;
    border: 5px solid rgba(255, 181, 62, 0.8);
    border-radius: 50%;
    text-align: center;
    line-height: 185px;
    font-weight: bold;
    box-sizing: border-box;
    transition: all 0.4s;
    &::before {
      content: "";
      display: block;
      width: 100%;
      height: 100%;
      position: absolute;
      left: 50%;
      top: 50%;
      border-radius: 50%;
      transform: translate(-50%, -50%);
      transition: all 0.3s;
      border: 2px solid rgba(255, 181, 62, 0.1);
    }
    &:hover {
      box-shadow: 0px 0px 100px 0px rgba(255, 181, 62, 1);

      &::before {
        content: "";
        width: 200%;
        height: 200%;
      }
    }
  }
}
</style>
