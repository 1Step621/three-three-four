<script setup lang="ts">
import { onMounted, ref, watch } from 'vue';
import ProgressBar from './components/ProgressBar.vue';
import Button from './components/Button.vue';
import ConfigPanel from './components/ConfigPanel.vue';
import Time from './components/Time.vue';

// config
const DEFAULT_CONFIG = {
  beepPoints: [{
    pos: 0.5,
    accent: false,
  }, {
    pos: 0.75,
    accent: true,
  }],
  tweak: 0,
  digits: 3,
};

const isConfigOpen = ref(false);
const config = ref(JSON.parse(localStorage.getItem('config') ?? JSON.stringify(DEFAULT_CONFIG)) as typeof DEFAULT_CONFIG);

watch(config, () => {
  localStorage.setItem('config', JSON.stringify(config.value));
}, { deep: true });

// date sync
type Res = {
  result: number;
  t1h: number;
  t1l: number;
  t2h: number;
  t2l: number;
  t3h: number;
  t3l: number;
  t4h: number;
  t4l: number;
};

const delta = ref(0);
const tick = ref(0);
const status = ref<'failed' | 'ntp' | 'cache'>();
const fixed = ref(new Date());
watch([delta, tick], ([delta,]) => {
  requestAnimationFrame(() => {
    const date = new Date();
    date.setTime(Date.now() + delta + config.value.tweak * 1000);
    fixed.value = date;
  });
});

const sync = async () => {
  const t1 = Date.now();
  const res: Res = await (await fetch("https://domisan.sakura.ne.jp/getjst.php")).json();
  const t4 = Date.now();

  let { t2l, t3l } = res;
  for (let i = 0; i < 32; i++) {
    t2l /= 2;
    t3l /= 2;
  }
  const t2 = (res.t2h + t2l) * 1000;
  const t3 = (res.t3h + t3l) * 1000;

  delta.value = ((t2 + t3) - (t1 + t4)) / 2;
  switch (res.result) {
    case 0:
      status.value = 'failed';
      break;
    case 1:
      status.value = 'ntp';
      break;
    case 2:
      status.value = 'cache';
      break;
    default:
      status.value = 'failed';
  }
};

// audio
const audioContext = new AudioContext();

const playSquareWave = (delay: number, accent: boolean) => {
  const oscillator = audioContext.createOscillator();
  oscillator.type = 'square';
  oscillator.frequency.setValueAtTime(accent ? 880 : 440, audioContext.currentTime);

  const gainNode = audioContext.createGain();
  gainNode.gain.setValueAtTime(1, audioContext.currentTime + delay);
  gainNode.gain.exponentialRampToValueAtTime(0.1, audioContext.currentTime + delay + 0.1);

  oscillator.connect(gainNode);
  gainNode.connect(audioContext.destination);

  oscillator.start(audioContext.currentTime + delay);
  oscillator.stop(audioContext.currentTime + delay + 0.1);
}

const startSound = () => {
  setInterval(() => {
    for (const point of config.value.beepPoints) {
      const milliseconds = fixed.value.getMilliseconds();
      const delay = (1000 - milliseconds + point.pos * 1000) % 1000;
      playSquareWave(delay / 1000, point.accent);
    }
  }, 1000);
}

onMounted(async () => {
  await sync();
  setInterval(sync, 1000 * 60 * 60);
  setInterval(() => {
    tick.value++;
  }, 1);
  startSound();
});
</script>

<template>
  <div :class="$style.configPanel" v-if="isConfigOpen">
    <ConfigPanel v-model="config" @close="isConfigOpen = false" />
  </div>
  <div :class="$style.main">
    <ProgressBar :class="$style.slider" :indicator="fixed.getMilliseconds() / 1000" :points="config.beepPoints" />
    <Time :date="fixed" :config="config" />
    <div :class="$style.meta">
      <span>
        status: {{ status ? (status === 'failed' ? 'failed' : 'success') : 'fetching...' }}
      </span>
      <span>
        source: {{ status ? status : 'fetching...' }}
      </span>
    </div>
    <div :class="$style.buttons">
      <Button @click="sync">Sync</Button>
      <Button @click="isConfigOpen = !isConfigOpen">Config</Button>
    </div>
  </div>
</template>

<style module>
.main {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  height: 100vh;
  gap: 10px;
  background-color: var(--black);
}

.configPanel {
  position: absolute;
  width: 100vw;
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
}

.slider {
  width: 80vw;
}

.meta {
  display: flex;
  justify-content: center;
  align-items: left;
  flex-direction: column;
}

.meta * {
  color: var(--gray);
  font-size: min(3vw, 20px);
}

.buttons {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 10px;

  position: absolute;
  bottom: 10px;
  right: 10px;
}
</style>
