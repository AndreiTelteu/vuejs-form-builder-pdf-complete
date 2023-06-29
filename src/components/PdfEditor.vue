<script setup lang="ts">
import { ref, reactive, onMounted, watchEffect, watch } from 'vue';
import { getDocument, GlobalWorkerOptions } from 'pdfjs-dist';

const src = new URL('pdfjs-dist/build/pdf.worker.js', import.meta.url);
GlobalWorkerOptions.workerSrc = src.toString();

const props = defineProps<{ url: string; fields: any }>();
let pdf = null;
const pdfpages = ref(0);
const pdfview = reactive([]);
const dragIndex = ref();

onMounted(async () => {
  let doc = await getDocument(props.url).promise;
  pdf = doc;
  pdfpages.value = doc.numPages;
});
watch(pdfview, async (pdfview2) => {
  if (pdfview2.length < pdfpages.value) return;
  let pages = [];
  for (let i = 1; i <= pdfpages.value; i++) {
    const page = await pdf.getPage(i);
    pages.push(page);
    await renderPage(pages[i - 1], pdfview2[i - 1]);
  }
});

const renderPage = async (page, canvas) => {
  console.log('rererenderrrr');
  if (!canvas) return;
  const scale = 1.5;
  const viewport = page.getViewport({ scale });
  // Support HiDPI-screens.
  const outputScale = window.devicePixelRatio || 1;

  const context = canvas?.getContext?.('2d');
  canvas.width = Math.floor(viewport.width * outputScale);
  canvas.height = Math.floor(viewport.height * outputScale);
  canvas.style.width = Math.floor(viewport.width) + 'px';
  canvas.style.height = Math.floor(viewport.height) + 'px';
  const transform =
    outputScale !== 1 ? [outputScale, 0, 0, outputScale, 0, 0] : null;

  // Render PDF page into canvas context
  const renderContext = {
    canvasContext: context,
    transform,
    viewport,
  };
  await page.render(renderContext).promise;
};

const events = {
  click(e, page) {
    // if (!props.fields[page]) props.fields[page] = [];
    // props.fields[page].push({
    //   left: Math.abs((e.layerX / e.target.offsetWidth) * 100).toFixed(3),
    //   top: Math.abs((e.layerY / e.target.offsetHeight) * 100).toFixed(3),
    // });
  },
  onMouseDown(e, page) {
    console.log('mousedown', dragIndex.value);
    e.preventDefault();
    if (!props.fields[page]) props.fields[page] = [];
    let top = Math.abs((e.layerY / e.target.offsetHeight) * 100).toFixed(3);
    let left = Math.abs((e.layerX / e.target.offsetWidth) * 100).toFixed(3);
    let newLength = props.fields[page].push({ left, top, width: 0, height: 0 });
    dragIndex.value = newLength - 1;
    console.log('mousedown', dragIndex.value, newLength);
  },
  onMouseMove(e, page) {
    e.preventDefault();
    if (dragIndex.value == null) return;
    console.log('mousemove', dragIndex.value);
    let top = Math.abs((e.layerY / e.target.offsetHeight) * 100).toFixed(3);
    let left = Math.abs((e.layerX / e.target.offsetWidth) * 100).toFixed(3);
    props.fields[page][dragIndex.value].width =
      left - props.fields[page][dragIndex.value].left;
    props.fields[page][dragIndex.value].height =
      top - props.fields[page][dragIndex.value].top;
  },
  onMouseUp(e, page) {
    console.log('mouseup', dragIndex.value);
    // e.preventDefault();
    if (dragIndex.value == null) return;
    dragIndex.value = null;
  },
  onMouseLeave(e, page) {
    this.onMouseUp(e, page);
  },
};
</script>

<template>
  <div
    v-for="index in pdfpages"
    :key="index"
    style="position: relative; width: 100%"
  >
    <canvas
      ref="pdfview"
      class="pdf-canvas"
      @click="events.click($event, index)"
      @mousedown="events.onMouseDown($event, index)"
      @mousemove="events.onMouseMove($event, index)"
      @mouseup="events.onMouseUp($event, index)"
      @mouseleave="events.onMouseLeave($event, index)"
    ></canvas>
    <span
      v-for="(pointer, pindex) in props.fields[index]"
      :key="pindex"
      :style="{
        position: 'absolute',
        top: pointer.top + '%',
        left: pointer.left + '%',
        width: pointer.width + '%',
        height: pointer.height + '%',
        border: '1px solid red',
        background: 'rgba(0,0,0,0.1)',
        pointerEvents: dragIndex ? 'none' : 'auto',
      }"
    >
    </span>
  </div>
</template>

<style scoped>
.pdf-canvas {
  position: relative;
  border: 1px solid black;
  min-width: 100%;
  max-width: 100%;
  height: auto !important;
}
</style>
