<script setup lang="ts">
import AppNavigation from "@/components/AppNavigation.vue";
</script>

<template>
  <div class="header">
    <div class="header__navigation">
      <AppNavigation></AppNavigation>
    </div>
  </div>
</template>

<style scoped>
.header {
  display: flex;
  justify-content: flex-start;
  margin-left: 10px;
  margin-right: 10px;
  border-bottom: 1px solid black;
}

.header__navigation {
  display: flex;
  align-items: baseline;
  margin-top: 10px;
  margin-bottom: 10px;
  margin-right: 30px;
}
</style>
