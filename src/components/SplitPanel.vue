<!--
 * @FileDescription: 该文件包含三个部分： pane-one、pane-trigger、pane-two表示分屏的上（左）、分割线、下（右）
 * @props：通过props可以设置direction、min、max、initPaneLengthPercent、initTriggerLength
 * @direction：横向分屏or纵向分屏，默认横向row
 * @min：分屏拖拽的最小百分比，默认10
 * @max：分屏拖拽的最大百分比，默认90
 * @initPaneLengthPercent：初始区域一的屏幕百分比，默认50
 * @initTriggerLength：初始分割线的大小，默认10px
 * @slot：分别在pane-one设置了具名插槽one,pane-two为two
-->
<script setup>
import { computed, ref } from "vue";
// 一、布局----------------------
// 父组件传来布局方向及区域最大最小比例
const props = defineProps({
  direction: {
    type: String,
    default: "row",
  },
  min: {
    type: Number,
    default: 10,
  },

  max: {
    type: Number,
    default: 90,
  },
  initPaneLengthPercent: {
    type: Number,
    default: 50,
  },

  initTriggerLength: {
    type: Number,
    default: 10,
  },
});
// 根据方向计算属性，如果是row就全都操作width，如果是column就全都操作height
const lengthType = computed(() => (props.direction === "row" ? "width" : "height"));

// 区域一和滑动器的宽高用响应式数据，以便拖拽时调整
const paneLengthPercent = ref(props.initPaneLengthPercent); //区域1高（宽）度%
const triggerLength = ref(props.initTriggerLength); //滑动器高（宽）度px
const triggerLeftOffset = ref(0); //鼠标距滑动器左(顶)侧偏移量
// 为了保证滑动器在区域1和区域2的正中间，区域1的宽度应该减去滑动器宽度的一半
const paneLengthValue = computed(
  () => `calc(${paneLengthPercent.value}% - ${triggerLength.value / 2 + "px"})`
);
const triggerLengthValue = computed(() => triggerLength.value + "px");

// 二、拖拽事件----------------
// 按下滑动器
const handleMouseDown = (e) => {
  document.addEventListener("mousemove", handleMouseMove);
  document.addEventListener("mouseup", handleMouseUp);
  if (props.direction === "row") {
    triggerLeftOffset.value = e.pageX - e.srcElement.getBoundingClientRect().left;
  } else {
    triggerLeftOffset.value = e.pageY - e.srcElement.getBoundingClientRect().top;
  }
};
// 监听模板引用,通过ref获取dom信息
const splitPane = ref(null);
// 按下滑动器后移动鼠标
const handleMouseMove = (e) => {
  // 根据鼠标移动的距离重新计算区域一的高（宽）百分比
  const clientRect = splitPane.value.getBoundingClientRect();
  let paneLengthPercentNew = 0;

  if (props.direction === "row") {
    const offset = e.pageX - clientRect.left - triggerLeftOffset.value + triggerLength.value / 2;
    paneLengthPercentNew = (offset / clientRect.width) * 100;
  } else {
    const offset = e.pageY - clientRect.top - triggerLeftOffset.value + triggerLength.value / 2;
    paneLengthPercentNew = (offset / clientRect.height) * 100;
  }

  if (paneLengthPercentNew < props.min) {
    paneLengthPercentNew = props.min;
  }
  if (paneLengthPercentNew > props.max) {
    paneLengthPercentNew = props.max;
  }

  paneLengthPercent.value = paneLengthPercentNew;
};

// 松开滑动器
const handleMouseUp = () => {
  document.removeEventListener("mousemove", handleMouseMove);
};
</script>

<template>
  <div class="split-pane" ref="splitPane" :style="{ flexDirection: direction }">
    <div class="pane pane-one" :style="lengthType + ':' + paneLengthValue">
      <slot name="one"></slot>
    </div>
    <div
      class="pane-trigger"
      :style="lengthType + ':' + triggerLengthValue"
      @mousedown="handleMouseDown"
    ></div>
    <div class="pane pane-two"><slot name="two"></slot></div>
  </div>
</template>
<style scoped lang="less">
.split-pane {
  // background: palegreen;
  height: 100%;
  display: flex;

  // .pane-one {
  //   background: palevioletred;
  // }

  .pane-trigger {
    background: white;
    user-select: none;
    cursor: row-resize;
  }

  .pane-two {
    flex: 1;
    // background: turquoise;
  }
}
</style>
