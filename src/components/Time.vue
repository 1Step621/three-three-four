<script lang="ts" setup>
import { computed } from 'vue';

const { date, config } = defineProps<{
  date: Date;
  config: {
    digits: number;
  };
}>();

const hour = computed(() => { return date.getHours().toString().padStart(2, '0'); });
const minute = computed(() => { return date.getMinutes().toString().padStart(2, '0'); });
const second = computed(() => { return date.getSeconds().toString().padStart(2, '0'); });
const millisecond = computed(() => { return date.getMilliseconds().toString().padStart(3, '0').slice(0, config.digits); });
</script>

<template>
  <span :class="$style.time">
    <span :class="$style.hour">{{ hour }}</span>
    <span :class="$style.punctuation">:</span>
    <span :class="$style.minute">{{ minute }}</span>
    <span :class="$style.punctuation">:</span>
    <span :class="$style.second">{{ second }}</span>
    <span :class="$style.punctuation">.</span>
    <span :class="$style.millisecond">{{ millisecond }}</span>
  </span>
</template>

<style module>
.time * {
  display: inline-flex;
  font-size: 10vw;
  font-weight: bold;
}

.time {
  color: var(--white);
}

.punctuation {
  color: var(--gray);
}
</style>
