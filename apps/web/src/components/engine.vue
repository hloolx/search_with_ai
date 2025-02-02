<script setup lang="ts">
import { ref, onMounted } from 'vue';
import { ISelectOptions } from '../interface';
import { useAppStore } from '../store';
import { useI18n } from 'vue-i18n';

const appStore = useAppStore();
const engines = ref<ISelectOptions[]>([]);
const engine = ref(appStore.engine);
const { t } = useI18n();

const onSelect = (val: any) => {
  appStore.updateEngine(val);
};

onMounted(() => {
  engines.value = [
    { name: 'SearXNG', value: 'SEARXNG' },
    { name: 'ChatGLM', value: 'CHATGLM' },
    { name: 'Google', value: 'GOOGLE' },
    { name: 'Bing', value: 'BING' },
    { name: 'Sogou', value: 'SOGOU' },
  ];
});

const getEngineName = (engineValue: string) => {
  return engines.value.find(e => e.value === engineValue)?.name || '';
};
</script>

<script lang="ts">
export default {
  name: 'SearchEngineSelect'
};
</script>

<template>
  <t-select
    v-model="engine"
    :placeholder="t('selectEngine')"
    @change="onSelect"
    :popup-props="{ 
      overlayClassName: 'select-popup rounded-xl',
      overlayInnerStyle: { 
        padding: '6px',
        borderRadius: '16px',
        border: '1px solid var(--td-border-level-2-color)',
        boxShadow: '0 3px 14px 2px rgba(0, 0, 0, 0.05)',
        maxHeight: '280px'
      }
    }"
  >
    <template #label>
      <div class="text-fade-container">
        <span class="text-sm text-gray-700 dark:text-gray-200">{{ getEngineName(engine) }}</span>
      </div>
    </template>
    <t-option
      v-for="item in engines"
      :key="item.value"
      :value="item.value"
    >
      <div class="flex items-center py-0.5">
        <span class="text-sm text-gray-700 dark:text-gray-200 whitespace-nowrap">{{ item.name }}</span>
      </div>
    </t-option>
  </t-select>
</template>

<style scoped>
.text-fade-container {
  position: relative;
  max-width: 100%;
  overflow: hidden;
  white-space: nowrap;
  mask-image: linear-gradient(to right, black 85%, transparent 100%);
  -webkit-mask-image: linear-gradient(to right, black 85%, transparent 100%);
}

:deep(.t-select) {
  --td-select-height: auto;
}

:deep(.t-input) {
  background: transparent;
  border: none;
  padding: 0;
}

:deep(.t-input__inner) {
  background: transparent;
}

:deep(.t-input__wrap) {
  padding: 0;
}

:deep(.t-select__input) {
  overflow: hidden;
}

:deep(.select-popup) {
  overflow: hidden !important;
}

:deep(.select-popup .t-option) {
  border-radius: 8px;
  margin: 2px;
  padding: 6px 12px;
  transition: all 0.2s ease;
}

:deep(.t-option:hover) {
  background-color: rgba(0, 0, 0, 0.04);
}

:deep(.t-is-selected) {
  background-color: rgba(0, 0, 0, 0.06);
  font-weight: 500;
}

:deep([theme-mode="dark"] .t-option:hover) {
  background-color: rgba(255, 255, 255, 0.08);
}

:deep([theme-mode="dark"] .t-is-selected) {
  background-color: rgba(255, 255, 255, 0.1);
}

:deep(.dark .select-popup) {
  background-color: var(--td-bg-color-container) !important;
  border-color: var(--td-border-level-2-color) !important;
}

:deep(.t-popup) {
  border-radius: 16px !important;
}
</style>
