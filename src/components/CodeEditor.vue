<template>
  <div class="editor-container">
    <div class="title">代码输入区</div>
    <textarea v-model="codeStore.rawCode" ></textarea>
  </div>
</template>

<script setup>
import { useCodeStore } from '@/global-data-store/codeStore';

import { ref } from 'vue'

const codeStore = useCodeStore()
</script>

<style scoped>
.editor-container {
  flex: 1;
  padding: 16px;
  background-color: var(--md-sys-color-background);
  color: var(--md-sys-color-on-background);
  display: flex;
  flex-direction: column;
}

.title {
  font: var(--md-sys-typescale-title-medium);
  color: var(--md-sys-color-on-surface-variant);
  margin-bottom: 8px;
  padding-left: 4px;
}

textarea {
  flex: 1;
  padding: 16px;
  border-radius: 8px;
  border: 1px solid var(--md-sys-color-outline);
  background-color: var(--md-sys-color-surface);
  color: var(--md-sys-color-on-surface);
  font-family: 'Roboto Mono', monospace; /* Keep monospace for code */
  font-size: 14px;
  line-height: 1.5;
  resize: none;
  overflow-y: auto;
  transition: border-color 0.2s;
}

textarea:focus {
  outline: none;
  border-color: var(--md-sys-color-primary);
  border-width: 2px;
  padding: 15px; /* Compensate for border width change to prevent jump */
}
</style>
