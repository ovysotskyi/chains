<script setup>
import {
  ref,
  reactive,
  onMounted,
  onBeforeUnmount,
  onUnmounted,
  useTemplateRef,
  computed,
  watchEffect,
} from "vue";

const CHAINS_NAME = "chains";

const chainId = Date.now();

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
  }, 10);
}

function openNewWindow() {
  window.open(window.location.href, "_blank", "width=400,height=400");
}

function setItem(item) {
  localStorage.setItem(CHAINS_NAME, JSON.stringify(item));
}

function getChains() {
  if (!localStorage.getItem(CHAINS_NAME)) {
    setItem([]);
  }

  return JSON.parse(localStorage.getItem(CHAINS_NAME));
}

function addChain() {
  let chains = getChains();
  let newLength = chains.push({ id: chainId });

  setItem(chains);

  console.log(newLength - 1);

  return newLength - 1;
}

function getChainId() {
  let chains = getChains();

  return chains.findIndex((chain) => chain.id === chainId);
}

function updateChain(posX, posY) {
  let cId = getChainId();
  if (cId === -1) {
    cId = addChain();
  }

  let chains = getChains();

  chains[cId].posX = posX;
  chains[cId].posY = posY;

  setItem(chains);
}

function removeChain() {
  let chains = getChains();

  chains = chains.filter((chain) => chain.id !== chainId);
  console.log(chainId, chains);

  setItem(chains);
}

onMounted(() => {
  window.addEventListener("resize", getWindowSize);
  window.addEventListener("beforeunload", removeChain);
  windowPosInterval.value = getWindowPos();

  createNeighborsChains();
});

const pointRef = useTemplateRef("pointRef");

const pointTelemetry = reactive({});
const angle = ref(0);
const neighborsChains = ref([]);

watchEffect(() => {
  let posX = windowTelemetry.innerWidth / 2 + windowTelemetry.posX;
  let posY =
    windowTelemetry.innerHeight / 2 +
    (windowTelemetry.height - windowTelemetry.innerHeight) +
    windowTelemetry.posY;

  pointTelemetry.value = {
    posX,
    posY,
  };

  updateChain(posX, posY);

  createNeighborsChains();
});

function createAngle(neighborChain) {
  let dx = neighborChain.posX - pointTelemetry.value.posX;
  let dy = neighborChain.posY - pointTelemetry.value.posY;

  let angle = (Math.atan2(dy, dx) * 180) / Math.PI;

  if (angle < 0) {
    angle = 360 + angle;
  }

  return angle;
}

function createNeighborsChains() {
  let chains = getChains();
  if (chains.length <= 1) neighborsChains.value = [];

  neighborsChains.value = chains
    .filter((ch) => ch.id !== chainId)
    .map((chain) => {
      return { id: chain.id, angle: createAngle(chain) };
    });
}

onMounted(() => {
  window.addEventListener("storage", () => createNeighborsChains());
});
</script>

<template>
  <button class="open-window" @click="openNewWindow">open new window</button>

  <div ref="pointRef" class="point">
    <div
      v-for="chain in neighborsChains"
      class="chain"
      :key="chain.id"
      :style="`transform: translate(0, -50%) rotate(${chain.angle}deg)`"
    ></div>
  </div>
</template>
