<template>
  <div>
    <slot />
    <div class="sidebar-area">
      <SpeedrunTimer />
    </div>
  </div>
</template>

<style>
:root {
  --sidebar-width: 280px;
}

/* Slidev's root slide container - reduce its width and let it center in the remaining space */
#slidev-player,
.slidev-layout,
.slidev-page {
  width: calc(100% - var(--sidebar-width)) !important;
}

.sidebar-area {
  position: fixed;
  top: 0;
  right: 0;
  width: var(--sidebar-width);
  height: 100%;
  z-index: 1000;
}
</style>
