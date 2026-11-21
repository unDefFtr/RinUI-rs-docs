<script setup lang="ts">
import { computed, onMounted, onUnmounted, ref } from 'vue'
import { useData } from 'vitepress'
import { VPButton } from 'vitepress/theme-without-fonts'
import { ChevronLeft, ChevronRight } from '@lucide/vue'

defineProps<{
  section: 'home' | 'guide'
  showCopy?: boolean
}>()

const { lang } = useData()
const current = ref(0)
const direction = ref(1)
const images = ['Screenshot.png', 'Screenshot-1.png', 'Screenshot-2.png', 'Screenshot-3.png', 'Screenshot-4.png']
const language = computed(() => lang.value.startsWith('zh') ? 'zh' : 'en')
const base = computed(() => `/assets/images/gallery-store${language.value === 'zh' ? '' : '/en'}`)
const guideLink = computed(() => `${lang.value.startsWith('zh') ? '/zh' : lang.value.startsWith('ja') ? '/ja' : ''}/guide/getting-started`)
const copy = computed(() => {
  if (lang.value.startsWith('zh')) return {
    title: 'rinui-rs Gallery',
    description: '运行 rinui-rs Gallery，通过交互式示例浏览 Fluent 风格 Qt Quick/QML 控件，查看组件说明、图标和源码入口。',
    download: '从源码运行 rinui-rs Gallery',
    source: '截图来自上游 RinUI Gallery，展示共用的 QML 控件。',
    previous: '上一张截图',
    next: '下一张截图',
    slide: (n: number) => `第 ${n} 张，共 ${images.length} 张`
  }
  if (lang.value.startsWith('ja')) return {
    title: 'rinui-rs Gallery',
    description: 'rinui-rs Gallery を実行して、Fluent スタイルの Qt Quick/QML コントロールをインタラクティブなサンプルで確認できます。コンポーネントの説明、アイコン、ソースコードへのリンクも閲覧できます。',
    download: 'rinui-rs Gallery をソースから実行',
    source: '画像は共通 QML コントロールを紹介する上流 RinUI Gallery のものです。',
    previous: '前のスクリーンショット',
    next: '次のスクリーンショット',
    slide: (n: number) => `${images.length} 枚中 ${n} 枚目`
  }
  return {
    title: 'rinui-rs Gallery',
    description: 'Run rinui-rs Gallery to explore Fluent-style Qt Quick/QML controls with interactive examples, component guides, icons, and source links.',
    download: 'Build rinui-rs Gallery',
    source: 'Screenshots from upstream RinUI Gallery illustrate the shared QML controls.',
    previous: 'Previous screenshot',
    next: 'Next screenshot',
    slide: (n: number) => `Screenshot ${n} of ${images.length}`
  }
})

let timer: ReturnType<typeof setInterval> | undefined
let prefersReducedMotion = false
const goTo = (index: number) => {
  direction.value = (index - current.value + images.length) % images.length > images.length / 2 ? -1 : 1
  current.value = index
}
const next = () => { direction.value = 1; current.value = (current.value + 1) % images.length }
const previous = () => { direction.value = -1; current.value = (current.value - 1 + images.length) % images.length }
const start = () => { if (!timer && !prefersReducedMotion) timer = setInterval(next, 6000) }
const stop = () => { if (timer) clearInterval(timer); timer = undefined }

onMounted(() => {
  prefersReducedMotion = window.matchMedia('(prefers-reduced-motion: reduce)').matches
  start()
})
onUnmounted(stop)
</script>

<template>
  <section class="gallery-carousel" :class="`gallery-carousel--${section}`" @mouseenter="stop" @mouseleave="start" @focusin="stop" @focusout="start" @keydown.left.prevent="previous" @keydown.right.prevent="next" aria-roledescription="carousel" :aria-label="copy.title">
    <template v-if="showCopy !== false">
      <h2>{{ copy.title }}</h2>
      <p>{{ copy.description }}</p>
    </template>
    <div class="gallery-carousel__stage" aria-live="off">
      <Transition :name="direction > 0 ? 'gallery-slide-forward' : 'gallery-slide-backward'">
        <img :key="current" :src="`${base}/${images[current]}`" :alt="copy.slide(current + 1)" loading="lazy">
      </Transition>
      <button class="gallery-carousel__arrow gallery-carousel__arrow--previous" type="button" :aria-label="copy.previous" @click="previous"><ChevronLeft :size="22" :stroke-width="2" aria-hidden="true" /></button>
      <button class="gallery-carousel__arrow gallery-carousel__arrow--next" type="button" :aria-label="copy.next" @click="next"><ChevronRight :size="22" :stroke-width="2" aria-hidden="true" /></button>
    </div>
    <div class="gallery-carousel__footer">
      <div class="gallery-carousel__dots" role="group" :aria-label="copy.title">
        <button v-for="(_, index) in images" :key="index" type="button" :class="{ active: current === index }" :aria-label="copy.slide(index + 1)" :aria-current="current === index ? 'true' : undefined" @click="goTo(index)" />
      </div>
      <div v-if="section === 'home'" class="gallery-carousel__actions">
        <VPButton class="gallery-carousel__download" size="medium" theme="brand" :text="copy.download" :href="guideLink" />
      </div>
    </div>
    <p class="gallery-carousel__source">{{ copy.source }}</p>
  </section>
</template>
