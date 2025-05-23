<script setup>
import useStore, { SecondStyle } from './store/index';
import { useElementSize } from '@vueuse/core';
import { ref, computed } from 'vue';
import { VDialog } from 'vuetify/components/VDialog';
import { VBtn } from 'vuetify/components/VBtn';
import { mdiCog } from '@mdi/js';
import Settings from './components/Settings.vue';

const store = useStore();

const showSetting = ref(false);

const elTime = ref();
const elWindow = ref();

const windowSize = useElementSize(elWindow);
const timeSize = useElementSize(elTime);
const scaleRatio = computed(() => Math.min(
  windowSize.width.value / timeSize.width.value,
  windowSize.height.value / timeSize.height.value
));

const wrapperStyle = computed(() => {
  return {
    '--background': store.preferences.background,
    '--foreground': store.preferences.foreground,
    '--font-family': store.preferences.fontFamily,
  }
})
const scaleStyle = computed(() => {
  return {
    '--scale': scaleRatio.value.toString(),
    '--top': (windowSize.height.value - timeSize.height.value * scaleRatio.value) / 2,
    '--left': (windowSize.width.value - timeSize.width.value * scaleRatio.value) / 2,
  };
});
const barStyle = computed(() => {
  switch (store.preferences.secondStyle) {
    case SecondStyle.FullscreenBar:
      return {
        class: ['bar', 'bar-full'],
        style: {
          '--color': store.preferences.colorProgress,
          '--progress': store.barProgress,
        }
      }
    case SecondStyle.UpperBar:
      return {
        class: ['bar', 'bar-upper'],
        style: {
          '--color': store.preferences.colorProgress,
          '--progress': store.barProgress,
        }
      }
    case SecondStyle.DigitIn:
      return {
        class: [],
        style: {},
      }
    case SecondStyle.Off:
      return {
        class: [],
        style: {},
      }
  }
});
</script>

<template>
  <div>
    <div class="full-wrapper" :style="wrapperStyle">
      <div class="d-flex justify-center">
        <v-dialog max-width="600" v-model="store.settingsOpen" v-if="showSetting">
          <template v-slot:activator="{ props }">
            <v-btn :prepend-icon="mdiCog" v-bind="props" color="primary">设置</v-btn>
          </template>
          <settings></settings>
        </v-dialog>
        <span class="title">{{ store.title }}</span>
      </div>
      <div class="time-window" ref="elWindow">
        <div class="main-time" ref="elTime" :style="scaleStyle" @dblclick="showSetting = !showSetting">{{ store.displayTime }}</div>
        <div :class="barStyle.class" :style="barStyle.style"></div>
      </div>
      <a style="text-align: center;" target="_blank" href="https://beian.miit.gov.cn">辽ICP备2025049633号-1</a>
    </div>
  </div>
</template>

<style scoped>
.full-wrapper {
  display: flex;
  flex-direction: column;
  width: 100vw;
  height: 100vh;
  background-color: var(--background);
  color: var(--foreground);
  font-family: var(--font-family);
}

.time-window {
  position: relative;
  flex-grow: 1;
}

.main-time {
  position: absolute;
  top: calc(var(--top) * 1px);
  left: calc(var(--left) * 1px);
  transform: scale(var(--scale, 1));
  transform-origin: left top;
  font-weight: bold;
}

.title {
  text-align: center;
  font-size: 36px;
  font-weight: bold;
  flex-grow: 1;
}

.bar {
  width: calc(var(--progress) * 100%);
  background-color: var(--color);
}

.bar.bar-full {
  height: 100%;
}

.bar.bar-upper {
  height: 16px;
}
</style>
