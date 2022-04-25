<template>
  <div class="jigsaw">
    <div class="jigsawBox">
      <div style="position: relative">
        <canvas id="image"></canvas>
        <canvas id="sliderBlock"></canvas>
      </div>
      <div class="slideBox">
        <div class="slideBar">拖动左边滑块完成上方拼图</div>
        <div class="slider" @mousedown.prevent="drag"></div>
      </div>
      <div class="tools">
        <div style="margin-right: 10px" @click="emit('close', false)">关闭</div>
        <div @click="canvasInit">刷新</div>
      </div>
    </div>
  </div>
</template>

<script lang="ts" name="Jigsaw" setup>
import { onMounted, reactive, ref, defineEmits } from "vue";

const emit = defineEmits(["close", "check"]);

let slider = reactive({
  mx: 0,
  bx: 0,
});
const canvasInit = (): void => {
  //生成指定区间的随机数
  const random = (min: number, max: number) => {
    return Math.floor(Math.random() * (max - min + 1) + min);
  };
  //x: 254, y: 109
  let mx = random(127, 244),
    bx = random(10, 128),
    y = random(10, 99);

  slider.mx = mx;
  slider.bx = bx;

  draw(mx, bx, y);
};

const draw = (mx = 200, bx = 20, y = 50): void => {
  let mainDom = document.querySelector("#image") as HTMLFormElement;
  let bg = mainDom.getContext("2d");
  let width = mainDom.width;
  let height = mainDom.height;

  let blockDom = document.querySelector("#sliderBlock") as HTMLFormElement;
  let block = blockDom.getContext("2d");
  //重新赋值，让canvas进行重新绘制
  blockDom.height = height;
  mainDom.height = height;

  let imgsrc: string;
  const n = Math.ceil(Math.random()*5)
  imgsrc = require(`@/assets/${n}.jpg`);
  let img = document.createElement("img") as HTMLImageElement;
  img.style.objectFit = "scale-down";
  img.src = imgsrc;
  img.onload = () => {
    bg.drawImage(img, 0, 0, width, height);
    block.drawImage(img, 0, 0, width, height);
  };
  let mainxy = { x: mx, y: y, r: 9 };
  let blockxy = { x: mx, y: y, r: 9 };
  drawBlock(bg, mainxy, "fill");
  drawBlock(block, blockxy, "clip");
};

const drawBlock = (
  ctx: CanvasRenderingContext2D,
  xy = { x: 254, y: 109, r: 9 },
  type: "fill" | "clip"
) => {
  let x = xy.x,
    y = xy.y,
    r = xy.r,
    w = 40;
  let PI = Math.PI;
  //绘制
  ctx.beginPath();
  //left
  ctx.moveTo(x, y);
  //top
  ctx.arc(x + (w + 5) / 2, y, r, -PI, 0, true);
  ctx.lineTo(x + w + 5, y);
  //right
  ctx.arc(x + w + 5, y + w / 2, r, 1.5 * PI, 0.5 * PI, false);
  ctx.lineTo(x + w + 5, y + w);
  //bottom
  ctx.arc(x + (w + 5) / 2, y + w, r, 0, PI, false);
  ctx.lineTo(x, y + w);
  ctx.arc(x, y + w / 2, r, 0.5 * PI, 1.5 * PI, true);
  ctx.lineTo(x, y);
  //修饰，没有会看不出效果
  ctx.lineWidth = 1;
  ctx.fillStyle = "rgba(255, 255, 255, 0.5)";
  ctx.strokeStyle = "rgba(255, 255, 255, 0.5)";
  ctx.stroke();
  ctx[type]();
  ctx.globalCompositeOperation = "xor";
  let sliderDom = document.querySelector("#sliderBlock") as HTMLFormElement;
  sliderDom.style.left = -1 * x + "px";
};
onMounted(() => {
  canvasInit();
});

const puzzle = ref(false);
const drag = (e: MouseEvent) => {
  console.log("鼠标按下", e);
  let dom = e.target as HTMLFormElement; //dom元素
  let sliderDom = document.querySelector("#sliderBlock") as HTMLFormElement; //滑块dom
  const downCoordinate = { x: e.x, y: e.y };
  const left = sliderDom.offsetLeft;

  //正确的滑块数据
  let checkx = left * -1;
  //x轴数据
  let x = 0;
  const move = (moveEV: { x: number; y: number }) => {
    x = moveEV.x - downCoordinate.x;
    //y = moveEV.y - downCoordinate.y;
    if (x >= 251 || x <= 0) return false;
    dom.style.left = x + "px";
    //dom.style.top = y + "px";
    sliderDom.style.left = left + x + "px";
  };

  const up = () => {
    document.removeEventListener("mousemove", move);
    document.removeEventListener("mouseup", up);
    dom.style.left = "";

    console.log(x, checkx, left);
    let max = checkx + 20;
    let min = checkx - 20;
    //允许正负误差
    if (max >= x && x >= min) {
      console.log("滑动解锁成功");
      puzzle.value = true;
      emit("close", true);
    } else {
      console.log("拼图位置不正确");
      alert('验证失败，请重试');
      puzzle.value = false;
      canvasInit();
    }
    emit('check', puzzle.value)
  };

  document.addEventListener("mousemove", move);
  document.addEventListener("mouseup", up);
};
</script>

<style scoped>
.jigsaw {
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.3);
  position: absolute;
  top: 0;
  left: 0;
  display: flex;
  justify-content: center;
  align-items: center;
}
.jigsawBox {
  width: 380px;
  height: 290px;
  padding: 20px;
  background: #fff;
  border-radius: 4px;
}
#image {
  width: 340px;
  height: 200px;
}
#sliderBlock {
  width: 340px;
  height: 200px;
  position: absolute;
  left: 0;
  top: 0;
  overflow: hidden;
}
.slideBox {
  width: 100%;
  height: 60px;
  position: relative;
  padding-top: 10px;
}
.slideBar {
  height: 40px;
  width: 100%;
  text-align: center;
  border-radius: 40px;
  background: #2e98ff;
  color: #fff;
  line-height: 40px;
}
.slider {
  width: 58px;
  height: 58px;
  position: absolute;
  top: 0;
  left: 0;
  border-radius: 30px;
  border: 1px solid #dfdfdf;
  background: #fff;
  cursor: pointer;
}
.tools {
  display: flex;
  justify-content: flex-start;
  align-items: center;
  font-size: 22px;
  cursor: pointer;
}
</style>
