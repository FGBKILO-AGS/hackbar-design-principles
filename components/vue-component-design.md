# Vue 组件设计 (Vue Component Design)

## 概述

HackBar 使用 Vue 3 和 Composition API 来构建用户界面。组件设计遵循单一职责原则，确保每个组件都有明确的功能和接口。

## 核心思想

1. **单一职责**：每个组件只负责一个特定的 UI 功能
2. **可复用性**：组件设计考虑在不同场景下的复用
3. **响应式数据**：充分利用 Vue 的响应式系统
4. **组合式 API**：使用 Composition API 组织组件逻辑

## 代码示例

### 基础组件结构

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
};

const onUrlInput = (event: Event) => {
  // 处理 URL 输入
};

const onBodyInput = (event: Event) => {
  // 处理 Body 输入
};

const sendRequest = () => {
  // 发送请求逻辑
};

const clearRequest = () => {
  // 清空请求逻辑
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
</style>
```

### 组合式函数 (Composables)

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
```

### 使用组合式函数的组件

```vue
<!-- components/RequestProcessor.vue -->
<template>
  <div>
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
  </div>
</template>

<script setup lang="ts">
import { ref, computed, watch } from 'vue';
import { useRequestProcessor } from '@/composables/useRequestProcessor';
import FormUrlencodedEditor from './FormUrlencodedEditor.vue';
import JsonEditor from './JsonEditor.vue';

const { setProcessor, processRequest } = useRequestProcessor();

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
  setProcessor(contentType);
};

// 监听请求变化并处理
watch(request, (newRequest) => {
  const processed = processRequest(newRequest);
  // 处理结果
}, { deep: true });
</script>
```

## 优势

1. **清晰的结构**：模板、逻辑和样式分离
2. **易于维护**：每个组件职责明确
3. **高度复用**：组件可以在不同地方复用
4. **响应式更新**：数据变化自动更新 UI

## 实践建议

1. 优先使用 `<script setup>` 语法
2. 合理划分组件粒度
3. 使用组合式函数封装可复用逻辑
4. 正确使用 props 和 emit 进行组件通信
5. 为组件编写清晰的文档和示例