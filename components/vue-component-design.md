# Vue 组件设计 (Vue Component Design)

## 原始定义

Vue 组件是 Vue.js 框架中的基本构建块，它们是可复用的 Vue 实例，具有预定义的选项。组件系统是 Vue 的核心特性之一，允许我们将用户界面拆分为独立可复用的代码块。

## 详细解释

Vue 组件设计是构建现代前端应用程序的关键要素。在 HackBar 项目中，我们使用 Vue 3 和 Composition API 来构建用户界面。组件设计遵循单一职责原则，确保每个组件都有明确的功能和接口。

### 核心概念

1. **组件化**：将用户界面拆分为独立的、可复用的组件
2. **响应式数据**：利用 Vue 的响应式系统管理组件状态
3. **生命周期**：组件具有明确的生命周期钩子
4. **通信机制**：组件之间通过 props 和 events 进行通信

### 在 HackBar 中的应用

HackBar 使用 Vue 3 和 Composition API 来构建用户界面。组件设计遵循单一职责原则，确保每个组件都有明确的功能和接口。

## 实施步骤

### 第一步：组件结构设计

设计组件的基本结构，包括模板、逻辑和样式。

```vue
<!-- components/RequestPanelBasic.vue -->
<template>
  <div class="request-panel">
    <v-card>
      <v-card-title>Request Panel</v-card-title>
      <v-card-text>
        <v-text-field
          v-model="request.method"
          label="Method"
          @change="onMethodChange"
        />
        <v-text-field
          v-model="request.url"
          label="URL"
          @input="onUrlInput"
        />
        <v-textarea
          v-model="request.body"
          label="Body"
          @input="onBodyInput"
        />
      </v-card-text>
      <v-card-actions>
        <v-btn @click="sendRequest">Send</v-btn>
        <v-btn @click="clearRequest">Clear</v-btn>
      </v-card-actions>
    </v-card>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive } from 'vue';
import type { HackbarRequest } from '@/types';

// 响应式数据
const request = reactive<HackbarRequest>({
  method: 'GET',
  url: '',
  headers: {},
  body: '',
  data: {}
});

// 事件处理函数
const onMethodChange = (event: Event) => {
  // 处理方法变更
  console.log('Method changed:', request.method);
};

const onUrlInput = (event: Event) => {
  // 处理 URL 输入
  console.log('URL input:', request.url);
};

const onBodyInput = (event: Event) => {
  // 处理 Body 输入
  console.log('Body input:', request.body);
};

const sendRequest = () => {
  // 发送请求逻辑
  console.log('Sending request:', request);
};

const clearRequest = () => {
  // 清空请求逻辑
  request.method = 'GET';
  request.url = '';
  request.body = '';
  request.headers = {};
  request.data = {};
};

// 暴露给父组件的接口
defineExpose({
  getRequest: () => request,
  setRequest: (newRequest: HackbarRequest) => {
    Object.assign(request, newRequest);
  }
});
</script>

<style scoped>
.request-panel {
  padding: 16px;
}

.v-card {
  margin-bottom: 16px;
}
</style>
```

### 第二步：创建组合式函数 (Composables)

将可复用的逻辑封装到组合式函数中。

```typescript
// composables/useRequestProcessor.ts
import { ref, computed } from 'vue';
import { ProcessorRegistry } from '@/processors';

export function useRequestProcessor() {
  const processorRegistry = new ProcessorRegistry();
  const currentProcessor = ref<RequestProcessor | null>(null);
  
  const setProcessor = (contentType: string) => {
    currentProcessor.value = processorRegistry.getProcessor(contentType) || null;
  };
  
  const processRequest = (request: HackbarRequest) => {
    if (currentProcessor.value) {
      return currentProcessor.value.process(request);
    }
    return {
      body: request.body,
      headers: request.headers
    };
  };
  
  return {
    setProcessor,
    processRequest,
    currentProcessor: computed(() => currentProcessor.value)
  };
}

// composables/useRequestHistory.ts
import { ref, computed } from 'vue';

export function useRequestHistory() {
  const history = ref<HackbarRequest[]>([]);
  const maxSize = 50; // 最大历史记录数
  
  const addToHistory = (request: HackbarRequest) => {
    // 添加时间戳
    const requestWithTimestamp = {
      ...request,
      timestamp: Date.now()
    };
    
    // 添加到历史记录开头
    history.value.unshift(requestWithTimestamp);
    
    // 限制历史记录数量
    if (history.value.length > maxSize) {
      history.value = history.value.slice(0, maxSize);
    }
    
    // 保存到本地存储
    saveToStorage();
  };
  
  const removeFromHistory = (index: number) => {
    history.value.splice(index, 1);
    saveToStorage();
  };
  
  const clearHistory = () => {
    history.value = [];
    saveToStorage();
  };
  
  const loadFromStorage = async () => {
    try {
      const result = await chrome.storage.local.get(['requestHistory']);
      history.value = result.requestHistory || [];
    } catch (error) {
      console.error('Failed to load request history:', error);
      history.value = [];
    }
  };
  
  const saveToStorage = async () => {
    try {
      await chrome.storage.local.set({ requestHistory: history.value });
    } catch (error) {
      console.error('Failed to save request history:', error);
    }
  };
  
  return {
    history: computed(() => history.value),
    addToHistory,
    removeFromHistory,
    clearHistory,
    loadFromStorage
  };
}
```

### 第三步：使用组合式函数的组件

在组件中使用组合式函数来管理复杂逻辑。

```vue
<!-- components/RequestProcessor.vue -->
<template>
  <div class="request-processor">
    <v-select
      v-model="selectedContentType"
      :items="availableContentTypes"
      label="Content Type"
      @update:model-value="onContentTypeChange"
    />
    
    <component 
      :is="currentProcessorComponent"
      v-model:request="request"
    />
    
    <div class="actions">
      <v-btn @click="processAndSend">Process & Send</v-btn>
      <v-btn @click="saveToHistory">Save to History</v-btn>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, watch } from 'vue';
import { useRequestProcessor } from '@/composables/useRequestProcessor';
import { useRequestHistory } from '@/composables/useRequestHistory';
import FormUrlencodedEditor from './FormUrlencodedEditor.vue';
import JsonEditor from './JsonEditor.vue';
import MultipartEditor from './MultipartEditor.vue';

// 使用组合式函数
const { setProcessor, processRequest } = useRequestProcessor();
const { addToHistory } = useRequestHistory();

const selectedContentType = ref('application/x-www-form-urlencoded');
const request = ref<HackbarRequest>({
  method: 'GET',
  url: '',
  headers: {},
  body: '',
  data: {}
});

const availableContentTypes = [
  'application/x-www-form-urlencoded',
  'application/json',
  'multipart/form-data'
];

const processorComponents = {
  'application/x-www-form-urlencoded': FormUrlencodedEditor,
  'application/json': JsonEditor,
  'multipart/form-data': MultipartEditor
};

const currentProcessorComponent = computed(() => {
  return processorComponents[selectedContentType.value] || 'div';
});

const onContentTypeChange = (contentType: string) => {
  selectedContentType.value = contentType;
  setProcessor(contentType);
  // 更新请求头
  request.value.headers['Content-Type'] = contentType;
};

// 监听请求变化并处理
watch(request, (newRequest) => {
  console.log('Request updated:', newRequest);
}, { deep: true });

const processAndSend = () => {
  try {
    // 处理请求
    const processedRequest = processRequest(request.value);
    
    // 发送请求（这里简化处理）
    console.log('Processed request:', processedRequest);
    
    // 实际发送请求的逻辑
    // sendRequest(processedRequest);
  } catch (error) {
    console.error('Failed to process request:', error);
  }
};

const saveToHistory = () => {
  addToHistory(request.value);
};
</script>

<style scoped>
.request-processor {
  padding: 16px;
}

.actions {
  margin-top: 16px;
  display: flex;
  gap: 8px;
}
</style>
```

### 第四步：组件通信和状态管理

实现组件间通信和状态管理。

```vue
<!-- components/RequestManager.vue -->
<template>
  <div class="request-manager">
    <RequestPanelBasic 
      ref="basicPanelRef"
      @request-change="handleRequestChange"
    />
    
    <RequestProcessor 
      :request="currentRequest"
      @update:request="updateRequest"
    />
    
    <PrettyRawResponse 
      :response="lastResponse"
      :language="responseLanguage"
    />
  </div>
</template>

<script setup lang="ts">
import { ref, reactive } from 'vue';
import RequestPanelBasic from './RequestPanelBasic.vue';
import RequestProcessor from './RequestProcessor.vue';
import PrettyRawResponse from './PrettyRawResponse.vue';

// 组件引用
const basicPanelRef = ref<InstanceType<typeof RequestPanelBasic> | null>(null);

// 响应式数据
const currentRequest = reactive<HackbarRequest>({
  method: 'GET',
  url: '',
  headers: {},
  body: '',
  data: {}
});

const lastResponse = ref<string>('');
const responseLanguage = ref<string>('http');

// 方法
const handleRequestChange = (request: HackbarRequest) => {
  // 处理来自基本面板的请求变更
  Object.assign(currentRequest, request);
};

const updateRequest = (newRequest: HackbarRequest) => {
  // 更新当前请求
  Object.assign(currentRequest, newRequest);
  
  // 如果基本面板存在，也更新它
  if (basicPanelRef.value) {
    basicPanelRef.value.setRequest(newRequest);
  }
};

// 暴露方法给父组件
defineExpose({
  getCurrentRequest: () => ({ ...currentRequest }),
  setResponse: (response: string, language: string = 'http') => {
    lastResponse.value = response;
    responseLanguage.value = language;
  }
});
</script>

<style scoped>
.request-manager {
  display: flex;
  flex-direction: column;
  gap: 16px;
}
</style>
```

## 优势

1. **清晰的结构**：模板、逻辑和样式分离
2. **易于维护**：每个组件职责明确
3. **高度复用**：组件可以在不同地方复用
4. **响应式更新**：数据变化自动更新 UI
5. **逻辑复用**：通过组合式函数复用逻辑

## 最佳实践

1. **优先使用 `<script setup>` 语法**：简化组件语法
2. **合理划分组件粒度**：避免组件过大或过小
3. **使用组合式函数封装可复用逻辑**：提高代码复用性
4. **正确使用 props 和 emit 进行组件通信**：保持组件间松耦合
5. **为组件编写清晰的文档和示例**：便于团队协作
6. **使用 TypeScript 提供类型安全**：减少运行时错误
7. **合理使用 scoped styles**：避免样式冲突

## 总结

Vue 组件设计是 HackBar 用户界面的核心。通过合理使用 Vue 3 的 Composition API 和组件化思想，我们能够构建出结构清晰、易于维护和扩展的用户界面。组件化设计不仅提高了开发效率，还增强了代码的可读性和可维护性。