# 请求处理机制 (Request Processing)

## 概述

HackBar 的核心功能是处理和发送 HTTP 请求。为了支持多种请求格式和处理方式，系统采用了灵活的请求处理机制，可以根据不同的 Content-Type 使用不同的处理器。

## 核心思想

1. **策略模式**：为不同类型的请求内容使用不同的处理策略
2. **可扩展性**：可以轻松添加新的请求处理方式
3. **统一接口**：所有处理器遵循统一的接口规范

## 代码示例

### 处理器接口定义

```typescript
// processors/processor.ts
export interface RequestProcessor {
  readonly contentType: string;
  process(request: HackbarRequest): ProcessResult;
  parse(body: string): any;
}

export interface HackbarRequest {
  method: string;
  url: string;
  headers: Record<string, string>;
  body: string;
  data: Record<string, string>;
}

export interface ProcessResult {
  body: string;
  headers: Record<string, string>;
}
```

### 具体处理器实现

```typescript
// processors/implementations/1-application-x-www-form-urlencoded.ts
export class FormUrlencodedProcessor implements RequestProcessor {
  readonly contentType = 'application/x-www-form-urlencoded';
  
  process(request: HackbarRequest): ProcessResult {
    const params = new URLSearchParams();
    for (const [key, value] of Object.entries(request.data)) {
      params.append(key, value);
    }
    
    return {
      body: params.toString(),
      headers: { 'Content-Type': this.contentType }
    };
  }
  
  parse(body: string): any {
    return Object.fromEntries(new URLSearchParams(body));
  }
}

// processors/implementations/3-application-json.ts
export class JsonProcessor implements RequestProcessor {
  readonly contentType = 'application/json';
  
  process(request: HackbarRequest): ProcessResult {
    return {
      body: JSON.stringify(request.data),
      headers: { 'Content-Type': this.contentType }
    };
  }
  
  parse(body: string): any {
    return JSON.parse(body);
  }
}
```

### 处理器注册和使用

```typescript
// processors/index.ts
export class ProcessorRegistry {
  private processors: RequestProcessor[] = [];
  
  constructor() {
    this.register(new FormUrlencodedProcessor());
    this.register(new JsonProcessor());
    // 注册其他处理器
  }
  
  register(processor: RequestProcessor) {
    this.processors.push(processor);
  }
  
  getProcessor(contentType: string): RequestProcessor | undefined {
    return this.processors.find(p => p.contentType === contentType);
  }
  
  processRequest(request: HackbarRequest): ProcessResult {
    const processor = this.getProcessor(request.headers['Content-Type']);
    if (processor) {
      return processor.process(request);
    }
    
    // 默认处理
    return {
      body: request.body,
      headers: request.headers
    };
  }
}
```

### 在组件中使用

```vue
<!-- RequestPanelBasic.vue -->
<template>
  <div>
    <v-text-field v-model="request.method" label="Method" />
    <v-text-field v-model="request.url" label="URL" />
    <v-textarea v-model="request.body" label="Body" />
    <!-- 其他UI元素 -->
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import { useProcessorRegistry } from '@/processors';

const request = ref({
  method: 'GET',
  url: '',
  headers: {},
  body: '',
  data: {}
});

const processorRegistry = useProcessorRegistry();

const processRequest = () => {
  const result = processorRegistry.processRequest(request.value);
  // 使用处理结果发送请求
};
</script>
```

## 优势

1. **支持多种格式**：可以处理不同类型的请求内容
2. **易于扩展**：添加新的内容类型只需要实现新的处理器
3. **代码清晰**：每种处理逻辑独立实现，便于维护
4. **复用性强**：处理器可以在不同场景下复用

## 实践建议

1. 为每种 Content-Type 实现专门的处理器
2. 处理器应该只关注数据的编码/解码，不涉及网络请求
3. 提供统一的处理器注册和查找机制
4. 对于不支持的格式提供默认处理方式