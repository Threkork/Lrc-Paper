<script setup>
import { ref } from 'vue';
import player from './components/player.vue';

import editPanel from './components/editPanel.vue';

</script>

<template>
  <div middle>
    <editPanel></editPanel>
  </div>

  <player></player>
</template>

<style>
.middle {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}
</style>