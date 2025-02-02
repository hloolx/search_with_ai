<template>
  <div id="home" class="flex size-full items-center justify-center dark:bg-black">
    <div class="mt-36 flex w-full flex-col gap-4 p-4 sm:-mt-28 lg:max-w-3xl xl:max-w-4xl">
      <div class="flex items-center justify-center gap-2">
        <img :src="logoUrl" class="w-10" />
        <span class="text-3xl font-bold text-gray-900 dark:text-gray-100">AI Search</span>
        <t-tag variant="light" class="text-xs text-gray-500">beta</t-tag>
      </div>
      <SearchInputBar :autofocus="true" :loading="false" @search="search" />
      
      <div class="my-2 flex flex-wrap items-center justify-center gap-4">
        <SearchMode />
        <SearCategory v-if="enableAdvanced" />
      </div>

      <!-- 热搜榜 -->
      <div class="mt-2 overflow-hidden rounded-lg border border-gray-200 bg-white/80 backdrop-blur-sm dark:border-gray-800 dark:bg-gray-900/80">
        <!-- 标题栏 -->
        <div class="flex items-center justify-between border-b border-gray-200 px-4 py-2.5 dark:border-gray-800">
          <span class="text-sm font-medium text-gray-700 dark:text-gray-200">实时热搜</span>
          <t-button
            variant="text"
            size="small"
            :loading="isLoading"
            :disabled="isLoading"
            class="!h-7 !w-7 !min-w-7 !p-0"
            @click="refreshHotSearch"
          >
            <template #icon>
              <RiRefreshLine 
                class="text-gray-500 transition-all" 
                :class="{ 'animate-spin': isLoading }"
              />
            </template>
          </t-button>
        </div>

        <!-- 平台切换 -->
        <div class="flex border-b border-gray-200 dark:border-gray-800">
          <div
            v-for="platform in platforms"
            :key="platform.value"
            class="relative flex-1 cursor-pointer px-4 py-2 text-center text-sm transition-all hover:bg-gray-50/50 dark:hover:bg-gray-800/50"
            :class="[
              currentPlatform === platform.value 
                ? 'text-blue-500 font-medium' 
                : 'text-gray-500 dark:text-gray-400'
            ]"
            @click="() => !isLoading && (currentPlatform = platform.value)"
          >
            {{ platform.label }}
            <div 
              class="absolute bottom-0 left-0 h-0.5 w-full bg-blue-500 opacity-0 transition-all duration-300"
              :class="{ 'opacity-100': currentPlatform === platform.value }"
            />
          </div>
        </div>
        
        <!-- 热搜列表 -->
        <div class="custom-scrollbar relative h-[400px] overflow-y-auto">
          <!-- 加载状态遮罩 -->
          <div 
            v-show="isLoading"
            class="absolute inset-0 z-10 flex items-center justify-center backdrop-blur-sm"
          >
            <t-loading size="small" />
          </div>
          
          <div
            v-for="(item, idx) in hotSearchList"
            :key="item.index"
            class="group flex cursor-pointer items-center justify-between border-b border-gray-100 px-4 py-2.5 transition-all hover:bg-gray-50/50 last:border-0 dark:border-gray-800 dark:hover:bg-gray-800/50"
            @click="onQuickSearch(item.title)"
          >
            <div class="flex items-center gap-3">
              <span 
                class="flex h-5 w-5 items-center justify-center text-xs font-medium"
                :class="[
                  idx < 3 
                    ? 'text-red-500' 
                    : 'text-gray-400'
                ]"
              >
                {{ idx + 1 }}
              </span>
              
              <span class="text-sm text-gray-700 transition-colors group-hover:text-gray-900 dark:text-gray-300 dark:group-hover:text-gray-100">
                {{ item.title }}
              </span>
            </div>
            
            <div class="flex items-center gap-3">
              <span class="text-xs text-gray-400">{{ formatHotValue(item.hot) }}</span>
              <RiSearch2Line 
                class="invisible text-gray-400 opacity-0 transition-all group-hover:visible group-hover:opacity-100" 
                size="14"
              />
            </div>
          </div>
        </div>
      </div>

      <div class="mt-4">
        <PageFooter />
      </div>
    </div>
  </div>
</template>

<script setup lang="tsx">
import { ref, watch, onMounted } from 'vue';
import router from '../router';
import { RiSearch2Line, RiRefreshLine } from '@remixicon/vue';
import { PageFooter, SearchInputBar, SearCategory, SearchMode } from '../components';
import logoUrl from '../assets/logo.png';
import { useI18n } from 'vue-i18n';
import { computed } from 'vue';
import { useAppStore } from '../store';

const { locale } = useI18n();
const appStore = useAppStore();

const enableAdvanced = computed(() => appStore.engine === 'SEARXNG');

const currentPlatform = ref('douyin');
const hotSearchList = ref<HotSearchItem[]>([]);

const platforms = [
  { value: 'douyin', label: '抖音', api: 'https://v2.xxapi.cn/api/douyinhot' },
  { value: 'weibo', label: '微博', api: 'https://v2.xxapi.cn/api/weibohot' },
  { value: 'baidu', label: '百度', api: 'https://v2.xxapi.cn/api/baiduhot' },
  { value: 'bilibili', label: 'B站', api: 'https://v2.xxapi.cn/api/bilibilihot' },
];

const quickly: Record<string, string[]> = {
  zh: [
    '刘亦菲《玫瑰的故事》',
    '怎么使用Ollama在本地部署大模型?',
    'llama3-70b需要什么硬件配置？',
    '小米su7体验怎么样？',
    '《庆余年2》大结局'
  ],
  en: [
    'What is LLM?',
    'What is RAG?',
    'How to use LLM in enterprise?'
  ],
  ptBR: [
    "O que é LLM?",
    "O que é RAG?",
    "Como utilizar LLM em empresas?"
  ]
};

const list = computed(() => {
  const key = locale.value;
  return quickly[key];
});

const search = (val: string) => {
  if (!val) {
    return;
  }
  router.push({
    name: 'SearchPage',
    query: {
      q: val
    }
  });
};

const onQuickSearch = (val: string) => {
  search(val);
};

interface HotSearchItem {
  title: string;
  hot: string | number;
  platform: 'weibo' | 'baidu' | 'douyin' | 'bilibili';
  index: number;
}

const isLoading = ref(false);

const refreshHotSearch = async () => {
  await fetchHotSearch(currentPlatform.value);
};

const fetchHotSearch = async (platform: string) => {
  const platformConfig = platforms.find(p => p.value === platform);
  if (!platformConfig) return;

  try {
    isLoading.value = true;
    const response = await fetch(platformConfig.api);
    const result = await response.json();
    if (result.code === 200) {
      hotSearchList.value = result.data.slice(0, 10).map((item: any, index: number) => {
        if (platform === 'bilibili') {
          return {
            title: item,
            hot: '热门',
            platform,
            index: index + 1
          };
        }
        return {
          title: platform === 'douyin' ? item.word : item.title,
          hot: platform === 'douyin' ? item.hot_value : item.hot,
          platform,
          index: index + 1
        };
      });
    }
  } catch (error) {
    console.error(`Failed to fetch ${platform} hot search:`, error);
  } finally {
    isLoading.value = false;
  }
};

const formatHotValue = (hot: string | number) => {
  if (typeof hot === 'number') {
    return hot > 10000 ? `${(hot / 10000).toFixed(1)}万` : hot;
  }
  return hot;
};

watch(currentPlatform, (newPlatform) => {
  fetchHotSearch(newPlatform);
});

onMounted(() => {
  fetchHotSearch(currentPlatform.value);
});
</script>

<script lang="tsx">
export default {
  name: 'HomePage'
};
</script>

<style scoped>
.custom-scrollbar {
  scrollbar-width: none;
  -ms-overflow-style: none;
}

.custom-scrollbar::-webkit-scrollbar {
  display: none;
}
</style>
