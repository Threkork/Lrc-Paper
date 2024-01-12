<script setup>
import { ref } from 'vue';
import player from './components/player.vue';

import editPanel from './components/editPanel.vue';

const err = ref(true)
</script>

<template>
  <div middle>
    <editPanel></editPanel>
  </div>

  <player></player>

  <div v-show="err" class="err">
    <p>
      error:页面暂不适配手机，真可惜，请用电脑使用吧
    </p>
    <button @click="err = !err">我确认我不是手机</button>
  </div>
</template>

<style>
.middle {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

@media (max-width: 768px) {
  .err {
    position: fixed;
    top: 0;
    left: 0;
    width: 100vw;
    height: 100vh;
    background-color: white;
    z-index: 6;
  }
}

@media (max-height: 768px) {
  .err {
    position: fixed;
    top: 0;
    left: 0;
    width: 100vw;
    height: 100vh;
    background-color: white;
    z-index: 6;
  }
}
</style>