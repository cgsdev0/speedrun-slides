<script setup>
import { ref, onUnmounted, watch } from "vue";
import { useNav } from "@slidev/client";

const { currentPage, currentSlideRoute } = useNav();

const elapsed = ref(0);
const running = ref(false);
const splits = ref([]);
const seenTitles = new Set();
let intervalId = null;
let startTime = 0;
let lastSplitTime = 0;
let hasStarted = false;

function formatParts(ms) {
  const totalMs = Math.max(0, Math.floor(ms));
  const minutes = Math.floor(totalMs / 60000);
  const seconds = Math.floor((totalMs % 60000) / 1000);
  const centis = Math.floor((totalMs % 1000) / 10);
  return {
    main: `${minutes.toString().padStart(2, "0")}:${seconds.toString().padStart(2, "0")}`,
    centis: centis.toString().padStart(2, "0"),
  };
}

function format(ms) {
  const totalMs = Math.max(0, Math.floor(ms));
  const minutes = Math.floor(totalMs / 60000);
  const seconds = Math.floor((totalMs % 60000) / 1000);
  const centis = Math.floor((totalMs % 1000) / 10);
  return `${minutes.toString().padStart(2, "0")}:${seconds.toString().padStart(2, "0")}.${centis.toString().padStart(2, "0")}`;
}

function getSlideTitle() {
  const meta = currentSlideRoute.value?.meta?.slide?.frontmatter;
  return (
    meta?.title ||
    currentSlideRoute.value?.meta?.slide?.title ||
    `Slide ${currentPage.value}`
  );
}

function start() {
  if (running.value) return;
  running.value = true;
  startTime = performance.now() - elapsed.value;
  intervalId = setInterval(() => {
    elapsed.value = performance.now() - startTime;
  }, 10);
}

function recordSplit(title) {
  if (seenTitles.has(title)) return;
  seenTitles.add(title);
  const diff = elapsed.value - lastSplitTime;
  splits.value.push({ title, time: elapsed.value, diff });
  lastSplitTime = elapsed.value;
}

watch(currentPage, () => {
  if (!hasStarted) {
    hasStarted = true;
    start();
    return;
  }
  if (!running.value) return;
  recordSplit(getSlideTitle());
});

onUnmounted(() => clearInterval(intervalId));
</script>

<template>
  <div class="speedrun-sidebar">
    <div class="time">
      <span class="time-main">{{ formatParts(elapsed).main }}</span>
      <span class="time-centis">.{{ formatParts(elapsed).centis }}</span>
    </div>

    <div class="splits" v-if="splits.length">
      <div v-for="(s, i) in splits" :key="i" class="split-row">
        <div class="split-title">{{ s.title }}</div>
        <div class="split-meta">
          <span class="split-diff">+{{ format(s.diff) }}</span>
          <span class="split-time">{{ format(s.time) }}</span>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.speedrun-sidebar {
  font-family: "Consolas", "Monaco", monospace;
  background: #111;
  color: #0f0;
  height: 100%;
  width: 100%;
  padding: 20px 16px;
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
  border-left: 1px solid #033;
}

.time {
  text-align: center;
  letter-spacing: 1px;
  flex-shrink: 0;
}

.time-main {
  font-size: 3.8rem;
  font-weight: bold;
  background: linear-gradient(to bottom, #7cffb2 0%, #2e9e5b 100%);
  -webkit-background-clip: text;
  background-clip: text;
  -webkit-text-fill-color: transparent;
  color: transparent;
}

.time-centis {
  font-size: 2.4rem;
  font-weight: bold;
  background: linear-gradient(to bottom, #7cffb2 0%, #2e9e5b 100%);
  -webkit-background-clip: text;
  background-clip: text;
  -webkit-text-fill-color: transparent;
  color: transparent;
}

.splits {
  margin-top: 20px;
  overflow-y: auto;
  flex: 1;
  display: flex;
  flex-direction: column;
  justify-content: flex-end;
}

.split-row {
  padding: 10px 0;
  border-bottom: 1px solid #1a1a1a;
}

.split-title {
  color: #fff;
  font-size: 0.95rem;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  margin-bottom: 4px;
}

.split-meta {
  display: flex;
  justify-content: space-between;
  font-size: 0.85rem;
}

.split-diff {
  color: #6f6;
}

.split-time {
  color: #fff;
}
</style>
