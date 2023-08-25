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
// 格子数量
const gridCount =
  (container.width / gridWidth) * (container.height / gridHeight);

// 寻找四个中心点
const centerPoint = [
  {
    x: container.width / 2 / gridWidth - 1,
    y: container.height / 2 / gridHeight - 1,
  },
  {
    x: container.width / 2 / gridWidth - 1 + 1,
    y: container.height / 2 / gridHeight - 1,
  },
  {
    x: container.width / 2 / gridWidth - 1,
    y: container.height / 2 / gridHeight - 1 + 1,
  },
  {
    x: container.width / 2 / gridWidth - 1 + 1,
    y: container.height / 2 / gridHeight - 1 + 1,
  },
];

const gridText = [
  {
    ring: 0,
    num: 1,
    px: 0,
    py: 0,
  },
  {
    ring: 0,
    num: 2,
    px: 100,
    py: 0,
  },
  {
    ring: 0,
    num: 3,
    px: 0,
    py: 100,
  },
  {
    ring: 0,
    num: 4,
    px: 100,
    py: 100,
  },
  {
    ring: 1,
    num: 1,
    px: -100,
    py: -100,
  },
  {
    ring: 1,
    num: 2,
    px: 0,
    py: -100,
  },
  {
    ring: 1,
    num: 3,
    px: 100,
    py: -100,
  },
  {
    ring: 1,
    num: 4,
    px: 200,
    py: -100,
  },
  {
    ring: 1,
    num: 5,
    px: 200,
    py: 0,
  },
  {
    ring: 1,
    num: 6,
    px: 200,
    py: 100,
  },
  {
    ring: 1,
    num: 7,
    px: 200,
    py: 200,
  },
  {
    ring: 1,
    num: 8,
    px: 100,
    py: 200,
  },
  {
    ring: 1,
    num: 9,
    px: 0,
    py: 200,
  },
  {
    ring: 1,
    num: 10,
    px: -100,
    py: 200,
  },
  {
    ring: 1,
    num: 11,
    px: -100,
    py: 100,
  },
  {
    ring: 1,
    num: 12,
    px: -100,
    py: 0,
  },
];
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
    // if (i == 1) {
    //   gridNum = 4;
    // }
    let old = null;
    for (let j = 0; j < gridNum; j++) {
      console.log(old);
      let x = 0;
      let y = 0;
      if (!old) {
        x = (i + 1) * gridWidth * -1 + gridWidth;
        y = (i + 1) * gridHeight * -1 + gridHeight;
      } else {
        x = old.px + gridWidth;
        y = old.py;
        if (j % (2 * (i + 1)) === 0) {
          x = old.px;
          y = old.py + gridHeight;
        }
        if (j % (gridNum - 1) === 0) {
          x = old.px - gridWidth;
          y = old.py;
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
  console.log(grids);
}
createdGrids(2);
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
      >
        {{ grid.ring }}-{{ grid.num }}
      </div>
      <div class="center">能量核心</div>
    </div>
  </div>
</template>

<style lang="scss" scoped>
.panel {
  width: 100vw;
  height: 100vh;
  overflow: hidden;
  .box {
    height: 100vh;
    transform: translate(30%, 30%);
    position: relative;
    .place {
      position: absolute;
      padding: 5px;
      border: 1px solid #fff;
      flex-shrink: 0;
      box-sizing: border-box;
      user-select: none;
    }
  }
  .center {
    position: absolute;
    width: 200px;
    height: 200px;
    border: 10px solid #fff;
    border-radius: 50%;
    text-align: center;
    line-height: 200px;
    font-weight: bold;
    box-sizing: border-box;
    &:hover {
      transform: scale(2);
    }
  }
}
</style>
