<script lang="ts" setup>
const emit = defineEmits<{
  (e: 'click'): void
}>();
</script>

<template>
  <button type="button" :class="$style.button" @click="emit('click')">
    <slot />
  </button>
</template>

<style module>
.button {
  appearance: none;
  font-family: var(--font-family);
  color: var(--white);
  background-color: var(--panel);
  border: none;
  border-radius: 5px;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 5px 10px;
  font-size: 15px;
  font-weight: bold;
  cursor: pointer;
}
</style>
