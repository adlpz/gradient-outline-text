<template>
  <q-page class="row column items-center justify-evenly">
    <div :style="`background: ${bgColor}`" class="full-width text-center" v-resize-text>
      <div class="text-gradient-outline">{{sampleText}}</div>
    </div>
    <component :is="'style'">body { background: {{bgColor}}; } {{css}}</component>
    <vue-gpickr v-model="gradient" />
    <pre><code>{{css}}</code></pre>
  </q-page>
</template>

<script lang="ts" setup>
import {ref, computed} from "vue";
import { VueGpickr, LinearGradient } from 'vue-gpickr';

const gradient = ref(new LinearGradient({
  angle: 0,
  stops: [
    ['#78e65c', 0],
    ['#127ac9', 1]
  ]
}));
const thickness = ref(1)
const bgColor = ref('#191325');
const sampleText = ref('Outline Gradient')

const gradientCss = computed(() => gradient.value.toString())
const shadowCss = computed(() => {
  const square = [
    [-1,-1],[-1,0],[-1,1],
    [ 0,-1],       [ 0,1],
    [ 1,-1],[ 1,0],[ 1,1]
  ]

  return square
      .map(([x, y]) => `${x*thickness.value}px ${y*thickness.value}px 0 #fff`)
      .join(', ')

})

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
  `
})
</script>
