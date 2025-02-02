<script lang="ts" setup>
import { ref, onMounted } from 'vue';
import { getModels } from '../api';
import { useAppStore } from '../store';
import { useI18n } from 'vue-i18n';

const appStore = useAppStore();
const model = ref(appStore.model);
const models = ref<string[]>([]);
const loading = ref(false);

const { t } = useI18n();

const onModelSelect = (val: any) => {
  appStore.updateModel(val);
};

onMounted(async () => {
  await listModels();
  if (appStore.model) {
    model.value = appStore.model;
  } else {
    model.value = models.value[0];
  }
  appStore.updateModel(model.value);
});

async function listModels () {
  loading.value = true;
  const res = await getModels();
  const keys = Object.keys(res);
  const values: string[] = [];
  keys.forEach((key) => {
    const vals = res[key] as string[];
    values.push(...vals.map(i => `${key}:${i}`));
  });

  models.value = values;
  loading.value = false;
}

// 格式化显示的模型名称
const formatModelName = (model: string) => {
  if (!model) return '';
  // 从 "provider:model" 格式中提取模型名称
  return model.split(':')[1] || model;
};

// 格式化显示的提供商名称
const formatProviderName = (model: string) => {
  if (!model) return '';
  return model.split(':')[0] || '';
};
</script>

<script lang="ts">
export default {
  name: 'ModelSelect'
};
</script>

<template>
  <t-select
    v-model="model"
    :loading="loading"
    :placeholder="t('selectModel')"
    @change="onModelSelect"
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
        <span class="text-sm text-gray-700 dark:text-gray-200">{{ formatModelName(model) }}</span>
      </div>
    </template>
    <t-option
      v-for="(item, index) in models"
      :key="index"
      :value="item"
    >
      <div class="flex flex-col py-0.5">
        <span class="text-sm text-gray-700 dark:text-gray-200">{{ formatModelName(item) }}</span>
        <span class="text-xs text-gray-400">{{ formatProviderName(item) }}</span>
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
  mask-image: linear-gradient(to right, black 80%, transparent 98%);
  -webkit-mask-image: linear-gradient(to right, black 80%, transparent 98%);
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

:deep(.t-option) {
  border-radius: 6px;
  margin: 2px 0;
}

:deep(.t-option:hover) {
  background-color: rgba(0, 0, 0, 0.04);
}

:deep(.t-is-selected) {
  background-color: rgba(0, 0, 0, 0.06);
}

:deep([theme-mode="dark"] .t-option:hover) {
  background-color: rgba(255, 255, 255, 0.08);
}

:deep([theme-mode="dark"] .t-is-selected) {
  background-color: rgba(255, 255, 255, 0.1);
}

:deep(.select-popup) {
  min-width: fit-content !important;
}

:deep(.select-popup .t-option) {
  border-radius: 8px;
  margin: 2px 0;
  padding: 8px 12px;
  width: auto;
}

:deep(.t-popup) {
  border-radius: 16px !important;
}
</style>
