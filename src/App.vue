<script setup>
import {
  ref,
  reactive,
  onMounted,
  onUnmounted,
  useTemplateRef,
  computed,
  watchEffect,
} from "vue";

const windowPosInterval = ref(null);

const screenSize = reactive({
  width: window.screen.width,
  height: window.screen.height,
});

const windowTelemetry = reactive({
  width: window.outerWidth,
  height: window.outerHeight,
  innerWidth: window.innerWidth,
  innerHeight: window.innerHeight,
  posX: window.screenX,
  posY: window.screenY,
});

function getWindowSize() {
  windowTelemetry.width = window.outerWidth;
  windowTelemetry.height = window.outerHeight;
  windowTelemetry.innerWidth = window.innerWidth;
  windowTelemetry.innerHeight = window.innerHeight;
}

function getWindowPos() {
  return setInterval(() => {
    windowTelemetry.posX = window.screenX;
    windowTelemetry.posY = window.screenY;
  }, 100);
}

onMounted(() => {
  window.addEventListener("resize", getWindowSize);
  windowPosInterval.value = getWindowPos();
});

onUnmounted(() => {
  window.removeEventListener("resize", getWindowSize);
  clearInterval(windowPosInterval.value);
});

const pointRef = useTemplateRef("pointRef");

const pointTelemetry = computed(() => {
  return {
    posX: windowTelemetry.innerWidth / 2 + windowTelemetry.posX,
    posY:
      windowTelemetry.innerHeight / 2 +
      (windowTelemetry.height - windowTelemetry.innerHeight) +
      windowTelemetry.posY,
  };
});

const angle = ref(0);

watchEffect(() => {
  let dx = screenSize.width / 2 - pointTelemetry.value.posX;
  let dy = screenSize.height / 2 - pointTelemetry.value.posY;

  angle.value = (Math.atan2(dy, dx) * 180) / Math.PI;

  if (angle.value < 0) {
    angle.value = 360 + angle.value;
  }
});

const chainStyle = computed(
  () => `transform: translate(0, -50%) rotate(${angle.value}deg)`,
);

function openNewWindow() {
  window.open(window.location.href, "_blank", "width=400,height=400");
}
</script>

<template>
  <button class="open-window" @click="openNewWindow">open new window</button>
  <div class="info">
    <table>
      <tbody>
        <tr>
          <th class="text-center" colspan="2">window size</th>
        </tr>
        <tr>
          <th class="text-left">width</th>
          <td>{{ windowTelemetry.width }}</td>
        </tr>
        <tr>
          <th class="text-left">height</th>
          <td>{{ windowTelemetry.height }}</td>
        </tr>
        <tr>
          <th class="text-left">inner width</th>
          <td>{{ windowTelemetry.innerWidth }}</td>
        </tr>
        <tr>
          <th class="text-left">inner height</th>
          <td>{{ windowTelemetry.innerHeight }}</td>
        </tr>
        <tr>
          <th class="text-center" colspan="2">window pos</th>
        </tr>
        <tr>
          <th class="text-left">x</th>
          <td>{{ windowTelemetry.posX }}</td>
        </tr>
        <tr>
          <th class="text-left">y</th>
          <td>{{ windowTelemetry.posY }}</td>
        </tr>
        <tr>
          <th class="text-center" colspan="2">point pos</th>
        </tr>
        <tr>
          <th class="text-left">x</th>
          <td>{{ pointTelemetry.posX }}</td>
        </tr>
        <tr>
          <th class="text-left">y</th>
          <td>{{ pointTelemetry.posY }}</td>
        </tr>
        <tr>
          <th class="text-center" colspan="2">screen</th>
        </tr>
        <tr>
          <th class="text-left">width</th>
          <td>{{ screenSize.width }}</td>
        </tr>
        <tr>
          <th class="text-left">height</th>
          <td>{{ screenSize.height }}</td>
        </tr>
        <tr>
          <th class="text-center" colspan="2">angle</th>
        </tr>
        <tr>
          <td>{{ angle }}</td>
        </tr>
      </tbody>
    </table>
  </div>

  <div ref="pointRef" class="point">
    <div class="chain" :style="chainStyle"></div>
  </div>
</template>
