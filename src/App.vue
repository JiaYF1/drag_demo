<script setup>
import SplitPane from "./components/SplitPanel.vue";
</script>

<template>
  <div class="page">
    <SplitPane :direction="'column'" :initPaneLengthPercent="60">
      <template #one>
        <div class="pane-one">paneOne</div>
      </template>
      <template #two>
        <div class="pane-two">paneTwo</div>
      </template>
    </SplitPane>
  </div>
</template>

<style scoped lang="less">
.page {
  height: 100vh;
  // background: #000;
  box-sizing: border-box;
}
.pane-one {
  height: 100%;
  background-color: palevioletred;
}
.pane-two {
  height: 100%;
  background-color: turquoise;
}
</style>
