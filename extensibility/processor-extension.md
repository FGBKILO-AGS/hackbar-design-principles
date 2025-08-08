# 处理器扩展 (Processor Extension)

## 概述

HackBar 的处理器系统允许扩展支持新的内容类型和数据格式。通过实现标准的处理器接口，开发者可以轻松添加对新格式的支持，使 HackBar 更加灵活和强大。

## 核心思想

1. **插件化架构**：通过插件机制支持新的处理器
2. **统一接口**：所有处理器遵循相同的接口规范
3. **动态注册**：运行时可以注册和使用新的处理器
4. **向后兼容**：新处理器不影响现有功能

## 代码示例

### 处理器接口定义

```typescript
// processors/processor.ts
export interface RequestProcessor {
  // 处理器支持的内容类型
  readonly contentType: string;
  
  // 处理请求数据
  process(request: HackbarRequest): ProcessResult;
  
  // 解析请求体
  parse(body: string): any;
  
  // 可选：验证请求数据
  validate?(request: HackbarRequest): boolean;
}

export interface HackbarRequest {
  id?: string;
  method: string;
  url: string;
  headers: Record<string, string>;
  body: string;
  data: Record<string, string>;
  timestamp?: number;
}

export interface ProcessResult {
  body: string;
  headers: Record<string, string>;
}
```

### 内置处理器实现

```typescript
// processors/implementations/1-application-x-www-form-urlencoded.ts
export class FormUrlencodedProcessor implements RequestProcessor {
  readonly contentType = 'application/x-www-form-urlencoded';
  
  process(request: HackbarRequest): ProcessResult {
    // 将数据转换为表单编码格式
    const params = new URLSearchParams();
    for (const [key, value] of Object.entries(request.data)) {
      params.append(key, value);
    }
    
    return {
      body: params.toString(),
      headers: {
        'Content-Type': this.contentType
      }
    };
  }
  
  parse(body: string): any {
    // 解析表单编码数据
    return Object.fromEntries(new URLSearchParams(body));
  }
  
  validate(request: HackbarRequest): boolean {
    // 验证请求数据
    return typeof request.data === 'object' && request.data !== null;
  }
}

// processors/implementations/3-application-json.ts
export class JsonProcessor implements RequestProcessor {
  readonly contentType = 'application/json';
  
  process(request: HackbarRequest): ProcessResult {
    // 将数据转换为JSON格式
    return {
      body: JSON.stringify(request.data),
      headers: {
        'Content-Type': this.contentType
      }
    };
  }
  
  parse(body: string): any {
    // 解析JSON数据
    try {
      return JSON.parse(body);
    } catch (error) {
      throw new Error('Invalid JSON format');
    }
  }
  
  validate(request: HackbarRequest): boolean {
    // 验证请求数据是否可以序列化为JSON
    try {
      JSON.stringify(request.data);
      return true;
    } catch {
      return false;
    }
  }
}
```

### 处理器注册系统

```typescript
// processors/index.ts
import type { RequestProcessor } from './processor';
import { FormUrlencodedProcessor } from './implementations/1-application-x-www-form-urlencoded';
import { JsonProcessor } from './implementations/3-application-json';
import { MultipartProcessor } from './implementations/4-multipart-form-data';

export class ProcessorRegistry {
  private processors: Map<string, RequestProcessor> = new Map();
  private defaultProcessor: RequestProcessor | null = null;
  
  constructor() {
    // 注册内置处理器
    this.register(new FormUrlencodedProcessor());
    this.register(new JsonProcessor());
    this.register(new MultipartProcessor());
    
    // 设置默认处理器
    this.defaultProcessor = this.processors.get('application/x-www-form-urlencoded') || null;
  }
  
  // 注册新的处理器
  register(processor: RequestProcessor): void {
    this.processors.set(processor.contentType, processor);
  }
  
  // 注销处理器
  unregister(contentType: string): boolean {
    return this.processors.delete(contentType);
  }
  
  // 获取特定类型的处理器
  getProcessor(contentType: string): RequestProcessor | undefined {
    return this.processors.get(contentType);
  }
  
  // 获取所有处理器
  getAllProcessors(): RequestProcessor[] {
    return Array.from(this.processors.values());
  }
  
  // 根据内容类型处理请求
  processRequest(request: HackbarRequest): ProcessResult {
    const contentType = request.headers['Content-Type'] || '';
    const processor = this.getProcessor(contentType);
    
    if (processor) {
      // 验证请求（如果处理器提供了验证方法）
      if (processor.validate && !processor.validate(request)) {
        throw new Error('Invalid request data for processor');
      }
      
      return processor.process(request);
    }
    
    // 使用默认处理器
    if (this.defaultProcessor) {
      return this.defaultProcessor.process(request);
    }
    
    // 不处理，返回原始数据
    return {
      body: request.body,
      headers: request.headers
    };
  }
  
  // 解析请求体
  parseBody(contentType: string, body: string): any {
    const processor = this.getProcessor(contentType);
    if (processor) {
      return processor.parse(body);
    }
    
    // 无法解析，返回原始字符串
    return body;
  }
}

// 创建全局处理器注册表
export const processorRegistry = new ProcessorRegistry();

// 提供组合式函数用于Vue组件
export function useProcessorRegistry() {
  return processorRegistry;
}
```

### 第三方处理器扩展示例

```typescript
// third-party-processors/xml-processor.ts
import type { RequestProcessor, HackbarRequest, ProcessResult } from '@/processors/processor';

export class XmlProcessor implements RequestProcessor {
  readonly contentType = 'application/xml';
  
  process(request: HackbarRequest): ProcessResult {
    // 将数据转换为XML格式
    let xml = '<?xml version="1.0" encoding="UTF-8"?>\n';
    xml += '<root>\n';
    
    for (const [key, value] of Object.entries(request.data)) {
      xml += `  <${key}><![CDATA[${value}]]></${key}>\n`;
    }
    
    xml += '</root>';
    
    return {
      body: xml,
      headers: {
        'Content-Type': this.contentType
      }
    };
  }
  
  parse(body: string): any {
    // 简化的XML解析实现
    // 在实际应用中，可能需要使用专门的XML解析库
    const result: Record<string, string> = {};
    const tagRegex = /<(\w+)>(.*?)<\/\1>/gs;
    
    let match;
    while ((match = tagRegex.exec(body)) !== null) {
      const [, tag, content] = match;
      result[tag] = content;
    }
    
    return result;
  }
  
  validate(request: HackbarRequest): boolean {
    // 验证数据是否适合XML格式
    for (const key in request.data) {
      // 检查键名是否包含非法字符
      if (!/^[a-zA-Z_][a-zA-Z0-9_-]*$/.test(key)) {
        return false;
      }
    }
    
    return true;
  }
}

// 注册XML处理器
import { processorRegistry } from '@/processors';

// 在应用初始化时注册
processorRegistry.register(new XmlProcessor());
```

### 在组件中使用处理器

```vue
<!-- components/RequestProcessorSelector.vue -->
<template>
  <div class="processor-selector">
    <v-select
      v-model="selectedContentType"
      :items="availableContentTypes"
      label="Content Type"
      @update:model-value="onContentTypeChange"
    />
    
    <component 
      :is="processorEditorComponent"
      v-model:data="requestData"
      :content-type="selectedContentType"
    />
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';
import { useProcessorRegistry } from '@/processors';
import FormUrlencodedEditor from './FormUrlencodedEditor.vue';
import JsonEditor from './JsonEditor.vue';
import MultipartEditor from './MultipartEditor.vue';

const props = defineProps<{
  modelValue: string;
  data: Record<string, string>;
}>();

const emit = defineEmits<{
  (e: 'update:modelValue', value: string): void;
  (e: 'update:data', value: Record<string, string>): void;
}>();

// 响应式数据
const selectedContentType = ref('application/x-www-form-urlencoded');
const requestData = ref<Record<string, string>>({});

// 处理器注册表
const processorRegistry = useProcessorRegistry();

// 计算属性
const availableContentTypes = computed(() => {
  return processorRegistry.getAllProcessors().map(p => p.contentType);
});

const processorEditorComponents: Record<string, any> = {
  'application/x-www-form-urlencoded': FormUrlencodedEditor,
  'application/json': JsonEditor,
  'multipart/form-data': MultipartEditor
};

const processorEditorComponent = computed(() => {
  return processorEditorComponents[selectedContentType.value] || 'div';
});

// 方法
const onContentTypeChange = (contentType: string) => {
  selectedContentType.value = contentType;
  emit('update:modelValue', contentType);
};

const processData = () => {
  // 使用处理器处理数据
  const request: HackbarRequest = {
    method: 'POST',
    url: '',
    headers: { 'Content-Type': selectedContentType.value },
    body: '',
    data: requestData.value
  };
  
  try {
    const result = processorRegistry.processRequest(request);
    return result;
  } catch (error) {
    console.error('Processor error:', error);
    throw error;
  }
};

// 监听器
watch(requestData, (newData) => {
  emit('update:data', newData);
}, { deep: true });

// 生命周期
onMounted(() => {
  selectedContentType.value = props.modelValue;
  requestData.value = { ...props.data };
});
</script>
```

## 优势

1. **可扩展性**：可以轻松添加新的内容类型支持
2. **模块化**：每个处理器独立实现，便于维护
3. **灵活性**：运行时可以动态注册和使用处理器
4. **兼容性**：新处理器不影响现有功能

## 实践建议

1. 为每种内容类型实现专门的处理器
2. 提供统一的处理器注册和管理机制
3. 实现处理器的验证功能确保数据完整性
4. 提供处理器的文档和使用示例
5. 考虑处理器的性能和安全性