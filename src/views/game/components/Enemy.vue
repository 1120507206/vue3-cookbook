<template>
  <div
    class="enemy"
    id="enemy"
    :style="{ width: enemy.width + 'px', height: enemy.height + 'px', right: enemy.right + 'px' }"
  >
    杆
  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch } from "vue";
import { onBeforeRouteLeave } from "vue-router";
import { ElNotification } from "element-plus";
import loadsh from "lodash";
import { useStore } from "vuex";
const store = useStore();

const brick = computed(() => store.state.game.brick);
const game = computed(() => store.state.game.game);
const figure = computed(() => store.state.game.figure);
const enemy = computed(() => store.state.game.enemy);

const enemyClone = ref();

const attack = ref(null);

watch(enemy.value, (val) => {
  if (val.currentLeft <= 31 && val.currentLeft >= -7) {
    // 碰撞
    if (figure.value.bottom <= enemy.value.height + brick.value.height) {
      successAttack();
    }
  }
});

// 过关失败：重置共计、分数清零、记录最高分
const successAttack = () => {
  cancelAnimationFrame(attack.value);
  attack.value = null;
  enemy.value.speed = enemyClone.value.speed;
  enemy.value.right = enemyClone.value.right;
  enemy.value.currentLeft = document.getElementById("enemy").offsetLeft;
  document.getElementById("enemy").style.right = enemy.value.right + "px";
  if (game.value.score > game.value.maxScore) {
    store.commit("game/setGameMaxScore", game.value.score);
  }
  store.commit("game/setGameScore", 0);
  store.commit("game/setGameStart", false);
};

// 过关成功：重置攻击、加分、加速
const resetAttack = () => {
  cancelAnimationFrame(attack.value);
  attack.value = null;
  enemy.value.right = enemyClone.value.right;
  document.getElementById("enemy").style.right = enemy.value.right + "px";
  store.commit("game/setGameScore", (game.value.score += 1));
  enemy.value.currentLeft = document.getElementById("enemy").offsetLeft;
  if (game.value.score === game.value.finalScore) {
    store.commit("game/setGameStart", false);
    enemy.value.speed = enemyClone.value.speed;
  } else {
    enemy.value.speed += enemy.value.speedAdd;
    ElNotification({
      title: "过关成功",
      message: "按空格开始下一关！",
      type: "success",
      duration: 3000,
    });
  }
};

// 敌人前进
const enemyPush = () => {
  const mapWidth = document.getElementById("map").offsetWidth;
  enemy.value.right += enemy.value.speed;
  enemy.value.currentLeft = mapWidth - enemy.value.right - enemy.value.width;
  document.getElementById("enemy").style.right = enemy.value.right + "px";
  attack.value = requestAnimationFrame(enemyPush);
  if (enemy.value.right >= mapWidth) {
    if (enemy.value.right !== enemyClone.value.right) {
      resetAttack();
    }
  }
};

const jumpFn = (e) => {
  if (e.keyCode === 32) {
    if (!game.value.start) return;
    if (attack.value) return;
    attack.value = requestAnimationFrame(enemyPush);
  }
};

onMounted(() => {
  let enemyDom = document.getElementById("enemy");
  if (!enemyDom) return;
  enemyClone.value = loadsh.cloneDeep(enemy.value);
  enemyDom.style.bottom = brick.value.height + "px";
  enemy.value.currentLeft = document.getElementById("enemy").offsetLeft;
  document.addEventListener("keydown", jumpFn);
});

onBeforeRouteLeave((to, from) => {
  if (attack.value) {
    cancelAnimationFrame(attack.value);
  }
  document.removeEventListener("keydown", jumpFn);
});
</script>

<style lang="scss" scoped>
#enemy {
  position: absolute;
  font-size: 20px;
  text-align: center;
  background-color: cadetblue;
}
</style>
