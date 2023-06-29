<script setup lang="ts">
import PdfEditor from './components/PdfEditor.vue';
import FieldEditor from './components/FieldEditor.vue';
import { reactive } from 'vue';

const fields = reactive({});
</script>

<template>
  <div
    style="
      width: 100vw;
      height: 100vh;
      display: flex;
      flex-direction: row;
      align-content: center;
      justify-content: center;
    "
  >
    <div style="overflow-x: auto; oferflow-y: hidden; width: 60%">
      <pdf-editor
        url="/VERAgene-SIF-ROMANIAN.pdf"
        :fields="fields"
      ></pdf-editor>
    </div>
    <div style="overflow: auto; width: 40%">
      <field-editor :fields="fields"></field-editor>
    </div>
  </div>
</template>

<style scoped></style>
