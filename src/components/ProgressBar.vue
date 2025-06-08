<script lang="ts" setup>
defineProps<{
  indicator: number;
  points: { pos: number; accent: boolean; }[];
}>();
</script>

<template>
  <div>
    <div :class="$style.progressBar" :style="{
      background: `linear-gradient(to right, var(--accent) ${indicator * 100}%, var(--white) ${indicator * 100}%)`
    }">
      <div v-for="point in points" :class="$style.point" :style="{
        left: `${point.pos * 100}%`,
        backgroundColor: point.pos <= indicator ? `var(--accent)` : `var(--white)`,
      }" />
    </div>
  </div>
</template>

<style module>
.progressBar {
  position: relative;
  width: 100%;
  height: 3px;
  border-radius: 10px;
}

.point {
  position: absolute;
  width: 10px;
  height: 10px;
  border-radius: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
}

@media screen and (max-width: 600px) {
  .progressBar {
    height: 2px;
  }

  .point {
    width: 7px;
    height: 7px;
  }
}
</style>
