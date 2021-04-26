<template>
  <main @contextmenu.prevent :style="{ height: height * 3 + 'px' }">
    <div
      :key="i"
      @touchstart="panDown"
      :ref="(el) => (pic[i] = el)"
      v-for="(_, i) in Array(9)"
      :style="{ width: width + 'px', height: height + 'px' }"
    />
  </main>

  <div :ref="(o) => (rulesRef = o)" class="rules" @click="hideRules">
    点击卡片互换位置
  </div>
</template>

<script>
import { onMounted, onUnmounted, reactive, toRefs } from "vue";
export default {
  setup() {
    const screenWidth = document.body.offsetWidth;
    const scale = screenWidth / (720 * 3);
    const width = scale * 720;
    const height = scale * 1080;
    const state = reactive({
      width,
      height,
      pic: [],
      mapRef: null,
      showShare: false,
      rulesRef: null,
    });

    const online = { break: [2, 5, 7, 4, 1, 8, 3, 6, 0], pass: "786150432" };
    // const dev = { break: [1, 0, 2, 3, 4, 5, 6, 7, 8], pass: "012345678" };
    const game = { ...online };

    let breakArr = []; // 当前位置信息
    let breakDoms = []; // 当前位置对应dom
    let selected = []; // 当前选中卡片
    let isCanPlay = false; // 是否初始化完成
    const positions = [
      [0, 0],
      [width, 0],
      [width * 2, 0],
      [0, height],
      [width, height],
      [width * 2, height],
      [0, height * 2],
      [width, height * 2],
      [width * 2, height * 2],
    ];

    async function finishAnim() {
      if (breakArr.join("") !== game.pass) return;
      isCanPlay = false;
      breakDoms.forEach((el) => {
        el.style.transition = "200ms";
        el.style.transform = "scale(.5)";
        el.style.border = "3vw solid #fff";
      });
      function back(indexes) {
        indexes.forEach((index) => {
          breakDoms[index].style.transition = "200ms";
          breakDoms[index].style.transform = "scale(1)";
          breakDoms[index].style.borderRadius = "0";
          breakDoms[index].style.borderWidth = "0";
        });
      }

      const timeout = 100;
      back([0]);
      await delay(timeout);
      back([1, 3]);
      await delay(timeout);
      back([2, 4, 6]);
      await delay(timeout);
      back([5, 7]);
      await delay(timeout);
      back([8]);
      const main = document.querySelector("main");
      main.style.transition = "400ms";
      main.classList.add("game-pass");
      main.style.transform = "scale(0.96) translateY(-50%)";
    }

    async function panDown(e) {
      if (!isCanPlay) return;
      const activeIndex = breakDoms.findIndex((el) => el === e.target);
      const target = e.target;
      const selectedIndex = selected.indexOf(activeIndex);

      if (selectedIndex !== -1) {
        breakDoms[activeIndex].classList.remove("active");
        selected.splice(selectedIndex, 1);
        return;
      }

      target.classList.add("active");

      selected.push(activeIndex);
      if (selected.length == 2) {
        isCanPlay = false;
        // 索引交换位置
        let temp = breakArr[selected[0]];
        breakArr[selected[0]] = breakArr[selected[1]];
        breakArr[selected[1]] = temp;
        // 元素交换位置
        let domTemp = breakDoms[selected[0]];
        breakDoms[selected[0]] = breakDoms[selected[1]];
        breakDoms[selected[1]] = domTemp;
        // 互换位置动画
        breakDoms[selected[0]].style.transition = "300ms";
        breakDoms[selected[0]].style.zIndex = "999";
        breakDoms[selected[0]].style.left = positions[selected[0]][0] + "px";
        breakDoms[selected[0]].style.top = positions[selected[0]][1] + "px";
        breakDoms[selected[1]].style.transition = "300ms";
        breakDoms[selected[1]].style.zIndex = "999";
        breakDoms[selected[1]].style.left = positions[selected[1]][0] + "px";
        breakDoms[selected[1]].style.top = positions[selected[1]][1] + "px";
        breakDoms.forEach((el) => {
          el.classList.remove("active");
        });
        await delay(300);
        // 重置元素
        breakDoms.forEach((el) => {
          el.classList.remove("active");
          el.style.transition = "";
          el.style.zIndex = "";
        });

        // 重置选中数组
        selected.length = 0;
        isCanPlay = true;
        // 检查是否完成
        finishAnim();
      }
    }
    function panUp() {
      if (!isCanPlay) return;
    }

    function panMove(e) {
      e.preventDefault();
    }

    async function initAnim() {
      const bgPosition = [
        "top left",
        "top center",
        "top right",
        "center left",
        "center center",
        "center right",
        "bottom left",
        "bottom center",
        "bottom right",
      ];
      // 初始化位置
      state.pic.forEach((el, i) => {
        el.style.left = positions[i][0] + "px";
        el.style.top = positions[i][1] + "px";
        el.style.backgroundPosition = bgPosition[i];
      });

      // 渐变
      await delay(10);
      const main = document.querySelector("main");
      main.style.opacity = 1;

      // 缩放
      await delay(500);
      main.style.zoom = "0.99";
      state.pic.forEach((el) => {
        el.style.transition = "300ms";
        el.style.transform = "scale(0.9,0.92)";
        el.style.boxShadow = "0 1vw 0.5vw rgba(0,0,0,0.4)";
        el.style.border = "0.5vw solid #fff";
        el.style.borderRadius = "2vw";
      });

      // 翻转
      await delay(200);
      state.pic.forEach((el) => {
        el.style.transition = "900ms";
        el.style.transform = "rotateY(180deg) scale(0.9,0.92)";
        el.style.backgroundPosition = "51% 73%";
        el.style.backgroundSize = "400% 400%";
      });

      // 集中
      await delay(400);
      state.pic.forEach((el, i) => {
        el.style.transition = "450ms ease-in";
        el.style.left = `33.3333%`;
        el.style.top = `33.3333%`;
      });

      // 打乱
      breakArr = game.break;

      // 分散
      await delay(450);
      state.pic.forEach((el, i) => {
        breakDoms[breakArr[i]] = el;
        el.style.transition = "450ms ease-out";
        el.style.left = positions[breakArr[i]][0] + "px";
        el.style.top = positions[breakArr[i]][1] + "px";
      });

      // 翻转回去
      await delay(300);
      state.pic.forEach((el, i) => {
        el.style.transition = "900ms";
        el.style.transform = "rotateY(0deg) scale(0.9,0.92)";
        el.style.backgroundPosition = bgPosition[i];
        el.style.backgroundSize = "300% 300%";
      });

      await delay(600);
      state.rulesRef.classList.add("rules-show");
      await delay(300);
      state.pic.forEach((el, i) => {
        el.style.transition = "";
      });
      isCanPlay = true;
      // end
    }

    function hideRules() {
      state.rulesRef.classList.remove("rules-show");
    }

    onMounted(() => {
      initAnim();
      window.addEventListener("touchend", panUp);
      window.addEventListener("touchcancel", panUp);
      window.addEventListener("touchmove", panMove, { passive: false });
    });

    onUnmounted(() => {
      window.removeEventListener("touchend", panUp);
      window.removeEventListener("touchcancel", panUp);
      window.removeEventListener("touchmove", panMove);
    });

    return { ...toRefs(state), panDown, hideRules };
  },
};
function delay(timeout = 0, callback = function () {}) {
  return new Promise(function (resolve, reject) {
    setTimeout(() => {
      try {
        callback();
        resolve();
      } catch (e) {
        reject(e);
      }
    }, timeout);
  });
}
</script>
