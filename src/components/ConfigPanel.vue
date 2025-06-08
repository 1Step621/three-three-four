<script lang="ts" setup>
defineEmits<{
  (e: 'close'): void;
}>();

const model = defineModel<{
  beepPoints: { pos: number; accent: boolean }[];
  tweak: number;
  digits: number;
}>({ required: true });
</script>

<template>
  <div :class="$style.configPanel">
    <div :class="$style.header">
      <span>Configuration</span>
      <button type="button" :class="$style.close" @click="$emit('close')">x</button>
    </div>
    <div :class="$style.content">
      <label>
        Beep Points:
        <div :class="$style.points">
          <div v-for="(_, index) in model.beepPoints" :key="index" :class="$style.point">
            <div>
              <input type="number" v-model="model.beepPoints[index].pos" :step="0.01" :min="0" :max="1" />s
            </div>
            <div>
              accent: <input type="checkbox" v-model="model.beepPoints[index].accent" />
            </div>
            <button type="button" @click="model.beepPoints.splice(index, 1)">x</button>
          </div>
        </div>
        <button type="button" @click="model.beepPoints.push({ pos: 1, accent: false })" :class="$style.add">
          Add Point
        </button>
      </label>
      <label>
        Tweak:
        +<input type="number" v-model="model.tweak" :step="0.01" :min="-1000000000000000" :max="1000000000000000" />s
      </label>
      <label>
        Digits:
        <input type="number" v-model="model.digits" :min="0" :max="3" />
      </label>
    </div>
  </div>
</template>

<style module>
.configPanel {
  position: absolute;
  z-index: 9999;
  width: 350px;
  max-width: 80%;
  padding: 10px;
  border-radius: 10px;

  background-color: var(--panel);

  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  gap: 10px;
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  width: 100%;
}

.header span {
  font-weight: bold;
  color: var(--white);
}

.content {
  display: flex;
  flex-direction: column;
  gap: 20px;
  width: 100%;
}

.points {
  display: flex;
  flex-direction: column;
  gap: 5px;
  margin-bottom: 5px;
}

.point {
  display: grid;
  grid-template-columns: 1fr 1fr auto;
  align-items: center;
}

.add {
  background-color: var(--accent);
  color: var(--white);
  padding: 5px;
  border-radius: 5px;
}

button {
  background: none;
  border: none;
  color: var(--white);
  font-size: 15px;
  cursor: pointer;
}

input[type="number"] {
  width: 80px;
  padding: 5px;
  appearance: none;

  border: 0;
  border-radius: 5px;
  font-size: 16px;
  background-color: var(--black);
  color: var(--white);
}

input[type="checkbox"] {
  accent-color: var(--accent);
}
</style>
