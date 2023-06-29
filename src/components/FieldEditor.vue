<script setup lang="ts">
import { ref, reactive, onMounted } from 'vue';

const props = defineProps<{ fields: any }>();
const visible = reactive({});

onMounted(async () => {});
</script>

<template>
  <div class="field-side">
    haha
    <button @click="Object.keys(fields).map((i) => delete fields[i])">
      clear</button
    ><br />
    {{ fields }}
    <div
      :class="{
        'field-page': true,
        'field-page-active': visible[page] == true,
      }"
      v-for="(items, page) in fields"
    >
      <div class="field-page-collapse" @click="visible[page] = !visible[page]">
        <span
          >Page: {{ page }}
          <small>(with {{ items.length }} fields)</small></span
        >
        <button>{{ visible[page] == true ? '▴' : '▾' }}</button>
      </div>
      <div class="field-page-items" v-show="visible[page] == true">
        <div :class="{ 'field-item': true }" v-for="(field, index) in items">
          #{{ index + 1 }}
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.field-side {
  padding: 20px;
}
.field-page-collapse {
  padding: 4px 8px;
  border-radius: 6px;
  background: #141414;
  cursor: pointer;
  display: flex;
  flex-direction: row;
  align-content: center;
  justify-content: space-between;
}
.field-page-active .field-page-collapse {
  background: #000;
}
.field-page-collapse button {
  width: 24px;
  border-radius: 100%;
  text-align: center;
  font-size: 22px;
  line-height: 0px;
}
.field-page-items {
  border: 1px solid #000;
  padding-top: 4px;
  margin-top: -4px;
  border-bottom-left-radius: 6px;
  border-bottom-right-radius: 6px;
  background: rgba(0, 0, 0, 0.2);
}
.field-item {
  margin: 10px;
}
</style>
