<template>
  <q-page class="row column items-center justify-evenly">
    <div
      class="full-width text-center preview-wrapper"
      :style="`background: ${bgColor}`"
    >
      <div v-resize-text="{ ratio: 0.9, minFontSize: 64, maxFontSize: 220 }">
        <div class="text-gradient-outline" v-html="sampleText" />
      </div>
    </div>
    <div class="q-pa-lg help-text text-h5 text-center">
      Generate a funky looking gradient outlined text in pure CSS.<br/>
      Tweak the knobs, then copy the resulting CSS.
    </div>
    <div class="row full-width items-top justify-center q-gutter-md q-mt-lg">
      <q-card>
        <q-card-section class="text-subtitle2">Gradient From</q-card-section>
        <q-separator />
        <q-card-section>
          <q-color no-header v-model="fromColor" class="colorPicker" />
        </q-card-section>
        <q-card-section v-if="fromTooDark" class="colorPickerError">
          This color is darker than the background. Weird artifacts may appear
          in the background. Change it to something lighter.
        </q-card-section>
      </q-card>
      <q-card>
        <q-card-section class="text-subtitle2">Gradient To</q-card-section>
        <q-separator />
        <q-card-section>
          <q-color no-header v-model="toColor" class="colorPicker" />
        </q-card-section>
        <q-card-section v-if="toTooDark" class="colorPickerError">
          This color is darker than the background. Weird artifacts may appear
          in the background. Change it to something lighter.
        </q-card-section>
      </q-card>
      <q-card>
        <q-card-section class="text-subtitle2">Background</q-card-section>
        <q-separator />
        <q-card-section>
          <q-color no-header v-model="bgColor" class="colorPicker" />
        </q-card-section>
      </q-card>
      <q-card style="min-width: 250px">
        <q-card-section class="text-subtitle2">Settings</q-card-section>
        <q-separator />
        <q-card-section class="row q-px-lg">
          <div class="text-center">Angle</div>
          <q-slider
            v-model="angle"
            :min="0"
            :max="360"
            :marker-labels="{ 0: '0°', 180: '180°', 360: '360°' }"
          />
          <div class="text-center">Thickness</div>
          <q-slider
            v-model="thickness"
            :min="1"
            :max="10"
            :marker-labels="{ 1: '1px', 5: '5px', 10: '10px' }"
          />
        </q-card-section>
      </q-card>
    </div>
    <q-card class="q-mt-lg css-card">
      <q-card-section class="text-subtitle2">Resulting CSS</q-card-section>
      <q-separator />
      <q-card-section>
        <pre><code>{{css}}</code></pre>
      </q-card-section>
    </q-card>
    <component :is="'style'">{{ css }}</component>

    <q-separator spaced />

    <div class="text-center q-pa-lg">
      Made in <a href="https://en.wikipedia.org/wiki/Valencia">Valencia</a> with
      ❤️ and 🥘 by <a href="https://prealfa.com">Adrià López</a>
    </div>
  </q-page>
</template>

<script lang="ts" setup>
import { ref, computed } from 'vue';
import { TinyColor } from '@ctrl/tinycolor';

const thickness = ref(1);
const bgColor = ref('#0c0c0d');
const fromColor = ref('#d712e6');
const toColor = ref('#227fc7');
const angle = ref(60);
const sampleText = ref('Outline<br>Gradient');

// Separate a color in three RGB channels
const toChannels = (color: string): [TinyColor, TinyColor, TinyColor] => {
  const colorObject = new TinyColor(color);
  const { r, g, b } = colorObject.toRgb();
  return [
    new TinyColor({ r, g: 0, b: 0 }),
    new TinyColor({ r: 0, g, b: 0 }),
    new TinyColor({ r: 0, g: 0, b }),
  ];
};

// Determine if a color will blend in 'darken' mode with the background
const isTooDark = (bg: string, fg: string) => {
  const bgChannels = toChannels(bg);
  const fgChannels = toChannels(fg);

  for (let i = 0; i < 3; i++) {
    if (bgChannels[i].getLuminance() > fgChannels[i].getLuminance()) {
      return true;
    }
  }

  return false;
};

const fromTooDark = computed(() => isTooDark(bgColor.value, fromColor.value));
const toTooDark = computed(() => isTooDark(bgColor.value, toColor.value));

const gradientCss = computed(
  () =>
    `linear-gradient(${angle.value}deg, ${fromColor.value} 0%, ${toColor.value} 100%)`
);
const shadowCss = computed(() => {
  const square = [
    [-1, -1],
    [-1, 0],
    [-1, 1],
    [0, -1],
    [0, 1],
    [1, -1],
    [1, 0],
    [1, 1],
  ];

  return square
    .map(([x, y]) => `${x * thickness.value}px ${y * thickness.value}px 0 #fff`)
    .join(',\n' + Array(17).fill(' ').join(''));
});

const css = computed(() => {
  return `
  .text-gradient-outline {
    color: ${bgColor.value};
    text-shadow: ${shadowCss.value};
    position: relative;
    display: inline-block;
    background-clip: text;
  }

  .text-gradient-outline:after {
    content: '';
    background: ${gradientCss.value};
    height: 100%;
    width: 100%;
    position: absolute;
    inset: 0;
    z-index: 1;
    mix-blend-mode: darken;
  }
  `;
});
</script>

<style scoped lang="scss">
.colorPicker {
  max-width: 200px;
}
.colorPickerError {
  text-align: center;
  color: red;
  max-width: 200px;
}
.css-card {
  width: 100%;
  max-width: 768px;
  pre {
    max-width: 100%;
    overflow-x: auto;
  }
}
.preview-wrapper {
  line-height: 1.1;
}
.help-text {
  max-width: 768px;
}
</style>
