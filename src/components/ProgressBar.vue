<template>
  <div
    ref="barWrap"
    class="aplayer-bar-wrap"
    @mousedown="onThumbMouseDown"
    @touchstart="onThumbTouchStart"
  >
    <div class="aplayer-bar">
      <div
        class="aplayer-loaded"
        :style="{ width: `${loadProgress * 100}%` }"
      />
      <div
        class="aplayer-played"
        :style="{ width: `${playProgress * 100}%`, background: theme }"
      >
        <span
          ref="thumb"
          class="aplayer-thumb"
          :style="{
            borderColor: theme,
            backgroundColor: thumbHovered ? theme : '#fff',
          }"
          @mouseover="thumbHovered = true"
          @mouseout="thumbHovered = false"
        >
          <span
            class="aplayer-loading-icon"
            :style="{ backgroundColor: theme }"
          >
            <Icon type="loading" />
          </span>
        </span>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from "vue";
import Icon from "./aplayer-icon.vue";
import { getElementViewLeft } from "../utils.js";

defineProps<{ loadProgress: number; playProgress: number; theme: string }>();
const emit = defineEmits<{
  (event: "drag-begin", percentage: number): void;
  (event: "dragging", percentage: number): void;
  (event: "drag-end", percentage: number): void;
}>();

const getPercentage = (clientX: number) => {
  const barWidth = barWrap.value!.clientWidth;
  const percentage = (clientX - getElementViewLeft(barWrap.value!)) / barWidth;

  return Math.min(Math.max(percentage, 0), 1);
};

const barWrap = ref<HTMLDivElement>();

const thumbHovered = ref(false);

const onDocumentMouseMove = ({ clientX }: MouseEvent): void => {
  emit("dragging", getPercentage(clientX));
};

const onThumbMouseDown = ({ clientX }: MouseEvent): void => {
  emit("drag-begin", getPercentage(clientX));

  document.addEventListener("mousemove", onDocumentMouseMove);
  document.addEventListener("mouseup", onDocumentMouseUp);
};

const onDocumentMouseUp = ({ clientX }: MouseEvent) => {
  emit("drag-end", getPercentage(clientX));

  document.removeEventListener("mousemove", onDocumentMouseMove);
  document.removeEventListener("mouseup", onDocumentMouseUp);
};

const onDocumentTouchMove = ({ changedTouches }: TouchEvent): void => {
  const touch = changedTouches[0];

  emit("dragging", getPercentage(touch.clientX));
};

const onDocumentTouchEnd = ({ changedTouches }: TouchEvent): void => {
  const touch = changedTouches[0];

  emit("drag-end", getPercentage(touch.clientX));
};

const onThumbTouchStart = ({ changedTouches }: TouchEvent) => {
  const touch = changedTouches[0];

  emit("drag-begin", getPercentage(touch.clientX));
  document.removeEventListener("touchmove", onDocumentTouchMove);
  document.removeEventListener("touchend", onDocumentTouchEnd);
};
</script>
<style lang="scss">
.aplayer-bar-wrap {
  margin: 0 0 0 5px;
  padding: 4px 0;
  cursor: pointer;
  flex: 1;

  .aplayer-bar {
    position: relative;
    height: 2px;
    width: 100%;
    background: #cdcdcd;

    .aplayer-loaded {
      position: absolute;
      left: 0;
      top: 0;
      bottom: 0;
      background: #aaa;
      height: 2px;
      transition: all 0.5s ease;

      will-change: width;
    }

    .aplayer-played {
      position: absolute;
      left: 0;
      top: 0;
      bottom: 0;
      height: 2px;
      transition: background-color 0.3s;
      will-change: width;

      .aplayer-thumb {
        position: absolute;
        top: 0;
        right: 5px;
        margin-top: -5px;
        margin-right: -10px;
        width: 10px;
        height: 10px;
        border: 1px solid;
        transform: scale(0.8);
        will-change: transform;
        transition: transform 300ms, background-color 0.3s, border-color 0.3s;
        border-radius: 50%;
        background: #fff;
        cursor: pointer;

        &:hover {
          transform: scale(1);
        }

        overflow: hidden;
        .aplayer-loading-icon {
          display: none;
          width: 100%;
          height: 100%;

          svg {
            position: absolute;
            animation: spin 1s linear infinite;
            fill: #ffffff;
          }
        }
      }
    }
  }
}

.aplayer-loading {
  .aplayer-bar-wrap .aplayer-bar .aplayer-thumb .aplayer-loading-icon {
    display: block;
  }

  .aplayer-info
    .aplayer-controller
    .aplayer-bar-wrap
    .aplayer-bar
    .aplayer-played
    .aplayer-thumb {
    transform: scale(1);
  }
}

@keyframes spin {
  0% {
    transform: rotate(0);
  }
  100% {
    transform: rotate(360deg);
  }
}
</style>
