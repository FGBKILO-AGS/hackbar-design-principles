# 处理器扩展 (Processor Extension)

## 原始定义

处理器扩展是一种软件设计模式，允许系统动态加载和使用处理特定数据格式或内容类型的组件。通过定义标准接口和实现机制，处理器扩展使应用程序能够支持新的数据格式而无需修改核心代码。

## 详细解释

HackBar 的处理器系统允许扩展支持新的内容类型和数据格式。通过实现标准的处理器接口，开发者可以轻松添加对新格式的支持，使 HackBar 更加灵活和强大。处理器扩展系统基于插件化架构，支持运行时注册、动态加载和热插拔等功能。

### 核心概念

1. **插件化架构**：通过插件机制支持新的处理器
2. **统一接口**：所有处理器遵循相同的接口规范
3. **动态注册**：运行时可以注册和使用新的处理器
4. **向后兼容**：新处理器不影响现有功能
5. **可扩展性**：支持第三方开发者创建自定义处理器

### 在 HackBar 中的应用

HackBar 的处理器系统允许扩展支持新的内容类型和数据格式。通过实现标准的处理器接口，开发者可以轻松添加对新格式的支持，使 HackBar 更加灵活和强大。

## 实施步骤

### 第一步：处理器接口定义

定义处理器的标准接口，确保所有处理器实现一致的功能。

```typescript
// processors/processor.ts
export interface RequestProcessor {
  // 处理器支持的内容类型
  readonly contentType: string;
  
  // 处理器名称
  readonly name: string;
  
  // 处理器版本
  readonly version: string;
  
  // 处理器描述
  readonly description: string;
  
  // 处理请求数据
  process(request: HackbarRequest): ProcessResult;
  
  // 解析请求体
  parse(body: string): any;
  
  // 验证请求数据
  validate(request: HackbarRequest): boolean;
  
  // 序列化数据为字符串
  serialize(data: any): string;
  
  // 获取处理器配置
  getConfig(): ProcessorConfig;
  
  // 设置处理器配置
  setConfig(config: ProcessorConfig): void;
}

// 请求数据结构
export interface HackbarRequest {
  id?: string;
  method: string;
  url: string;
  headers: Record<string, string>;
  body: string;
  data: Record<string, string>;
  timestamp?: number;
}

// 处理结果结构
export interface ProcessResult {
  body: string;
  headers: Record<string, string>;
}

// 处理器配置接口
export interface ProcessorConfig {
  [key: string]: any;
}

// 处理器元数据
export interface ProcessorMetadata {
  id: string;
  name: string;
  version: string;
  description: string;
  contentType: string;
  author?: string;
  website?: string;
}
```

### 第二步：内置处理器实现

实现系统内置的处理器，支持常见的内容类型。

```typescript
// processors/implementations/1-application-x-www-form-urlencoded.ts
export class FormUrlencodedProcessor implements RequestProcessor {
  readonly contentType = 'application/x-www-form-urlencoded';
  readonly name = 'Form URL Encoded Processor';
  readonly version = '1.0.0';
  readonly description = '处理 application/x-www-form-urlencoded 格式的请求';
  
  private config: ProcessorConfig = {
    encodeSpacesAsPlus: true,
    sortKeys: false
  };
  
  process(request: HackbarRequest): ProcessResult {
    try {
      // 将数据转换为表单编码格式
      const params = new URLSearchParams();
      
      // 根据配置决定是否排序键
      const keys = this.config.sortKeys 
        ? Object.keys(request.data).sort() 
        : Object.keys(request.data);
      
      for (const key of keys) {
        const value = request.data[key];
        params.append(key, value);
      }
      
      // 根据配置决定是否将空格编码为加号
      let body = params.toString();
      if (this.config.encodeSpacesAsPlus) {
        body = body.replace(/%20/g, '+');
      }
      
      return {
        body,
        headers: {
          'Content-Type': this.contentType
        }
      };
    } catch (error) {
      throw new Error(`Failed to process form data: ${error instanceof Error ? error.message : 'Unknown error'}`);
    }
  }
  
  parse(body: string): any {
    try {
      // 解析表单编码数据
      // 将加号转换为空格
      const normalizedBody = body.replace(/\+/g, ' ');
      return Object.fromEntries(new URLSearchParams(normalizedBody));
    } catch (error) {
      throw new Error(`Failed to parse form data: ${error instanceof Error ? error.message : 'Unknown error'}`);
    }
  }
  
  validate(request: HackbarRequest): boolean {
    // 验证请求数据
    if (!request.data || typeof request.data !== 'object') {
      return false;
    }
    
    // 验证所有键值都是字符串
    for (const [key, value] of Object.entries(request.data)) {
      if (typeof key !== 'string' || typeof value !== 'string') {
        return false;
      }
    }
    
    return true;
  }
  
  serialize(data: any): string {
    if (typeof data !== 'object' || data === null) {
      throw new Error('Data must be an object');
    }
    
    const params = new URLSearchParams();
    for (const [key, value] of Object.entries(data)) {
      if (typeof key === 'string' && typeof value === 'string') {
        params.append(key, value);
      }
    }
    
    return params.toString();
  }
  
  getConfig(): ProcessorConfig {
    return { ...this.config };
  }
  
  setConfig(config: ProcessorConfig): void {
    this.config = { ...this.config, ...config };
  }
}

// processors/implementations/3-application-json.ts
export class JsonProcessor implements RequestProcessor {
  readonly contentType = 'application/json';
  readonly name = 'JSON Processor';
  readonly version = '1.0.0';
  readonly description = '处理 application/json 格式的请求';
  
  private config: ProcessorConfig = {
    indent: 2,
    sortKeys: false
  };
  
  process(request: HackbarRequest): ProcessResult {
    try {
      // 将数据转换为JSON格式
      const jsonString = this.serialize(request.data);
      
      return {
        body: jsonString,
        headers: {
          'Content-Type': this.contentType
        }
      };
    } catch (error) {
      throw new Error(`Failed to process JSON data: ${error instanceof Error ? error.message : 'Unknown error'}`);
    }
  }
  
  parse(body: string): any {
    try {
      return JSON.parse(body);
    } catch (error) {
      throw new Error(`Invalid JSON format: ${error instanceof Error ? error.message : 'Unknown error'}`);
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
  
  serialize(data: any): string {
    try {
      if (this.config.sortKeys) {
        // 如果需要排序键，先排序
        const sortedData = this.sortObjectKeys(data);
        return JSON.stringify(sortedData, null, this.config.indent);
      } else {
        return JSON.stringify(data, null, this.config.indent);
      }
    } catch (error) {
      throw new Error(`Failed to serialize data to JSON: ${error instanceof Error ? error.message : 'Unknown error'}`);
    }
  }
  
  private sortObjectKeys(obj: any): any {
    if (typeof obj !== 'object' || obj === null) {
      return obj;
    }
    
    if (Array.isArray(obj)) {
      return obj.map(item => this.sortObjectKeys(item));
    }
    
    const sortedObj: any = {};
    Object.keys(obj).sort().forEach(key => {
      sortedObj[key] = this.sortObjectKeys(obj[key]);
    });
    
    return sortedObj;
  }
  
  getConfig(): ProcessorConfig {
    return { ...this.config };
  }
  
  setConfig(config: ProcessorConfig): void {
    this.config = { ...this.config, ...config };
  }
}
```

### 第三步：处理器注册系统实现

实现处理器的注册、管理和发现机制。

```typescript
// processors/index.ts
import type { RequestProcessor, ProcessorMetadata } from './processor';
import { FormUrlencodedProcessor } from './implementations/1-application-x-www-form-urlencoded';
import { JsonProcessor } from './implementations/3-application-json';
import { MultipartProcessor } from './implementations/4-multipart-form-data';

export class ProcessorRegistry {
  private processors: Map<string, RequestProcessor> = new Map();
  private metadata: Map<string, ProcessorMetadata> = new Map();
  private defaultProcessor: RequestProcessor | null = null;
  private observers: Array<(processor: RequestProcessor) => void> = [];
  
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
    // 检查是否已存在相同内容类型的处理器
    if (this.processors.has(processor.contentType)) {
      console.warn(`Processor for content type ${processor.contentType} already exists. Overriding.`);
    }
    
    // 注册处理器
    this.processors.set(processor.contentType, processor);
    
    // 注册元数据
    const metadata: ProcessorMetadata = {
      id: this.generateProcessorId(processor),
      name: processor.name,
      version: processor.version,
      description: processor.description,
      contentType: processor.contentType
    };
    
    this.metadata.set(processor.contentType, metadata);
    
    // 通知观察者
    this.notifyObservers(processor);
    
    console.log(`Processor registered: ${processor.name} (${processor.contentType})`);
  }
  
  // 注销处理器
  unregister(contentType: string): boolean {
    const processor = this.processors.get(contentType);
    if (!processor) {
      return false;
    }
    
    this.processors.delete(contentType);
    this.metadata.delete(contentType);
    
    // 如果注销的是默认处理器，重置默认处理器
    if (this.defaultProcessor?.contentType === contentType) {
      this.defaultProcessor = this.processors.get('application/x-www-form-urlencoded') || null;
    }
    
    console.log(`Processor unregistered: ${processor.name} (${contentType})`);
    return true;
  }
  
  // 获取特定类型的处理器
  getProcessor(contentType: string): RequestProcessor | undefined {
    return this.processors.get(contentType);
  }
  
  // 获取所有处理器
  getAllProcessors(): RequestProcessor[] {
    return Array.from(this.processors.values());
  }
  
  // 获取处理器元数据
  getProcessorMetadata(contentType: string): ProcessorMetadata | undefined {
    return this.metadata.get(contentType);
  }
  
  // 获取所有处理器元数据
  getAllProcessorMetadata(): ProcessorMetadata[] {
    return Array.from(this.metadata.values());
  }
  
  // 根据内容类型处理请求
  processRequest(request: HackbarRequest): ProcessResult {
    const contentType = request.headers['Content-Type'] || '';
    const processor = this.getProcessor(contentType);
    
    if (processor) {
      // 验证请求（如果处理器提供了验证方法）
      if (!processor.validate(request)) {
        throw new Error('Invalid request data for processor');
      }
      
      return processor.process(request);
    }
    
    // 使用默认处理器
    if (this.defaultProcessor) {
      if (!this.defaultProcessor.validate(request)) {
        throw new Error('Invalid request data for default processor');
      }
      
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
  
  // 设置默认处理器
  setDefaultProcessor(contentType: string): boolean {
    const processor = this.getProcessor(contentType);
    if (processor) {
      this.defaultProcessor = processor;
      return true;
    }
    return false;
  }
  
  // 获取默认处理器
  getDefaultProcessor(): RequestProcessor | null {
    return this.defaultProcessor;
  }
  
  // 注册观察者
  addObserver(observer: (processor: RequestProcessor) => void): void {
    this.observers.push(observer);
  }
  
  // 移除观察者
  removeObserver(observer: (processor: RequestProcessor) => void): void {
    const index = this.observers.indexOf(observer);
    if (index !== -1) {
      this.observers.splice(index, 1);
    }
  }
  
  // 通知观察者
  private notifyObservers(processor: RequestProcessor): void {
    this.observers.forEach(observer => {
      try {
        observer(processor);
      } catch (error) {
        console.error('Error in processor registry observer:', error);
      }
    });
  }
  
  // 生成处理器ID
  private generateProcessorId(processor: RequestProcessor): string {
    return `${processor.contentType.replace(/\//g, '-')}-${processor.version}`;
  }
  
  // 清理资源
  cleanup(): void {
    this.processors.clear();
    this.metadata.clear();
    this.observers = [];
    this.defaultProcessor = null;
  }
}

// 创建全局处理器注册表
export const processorRegistry = new ProcessorRegistry();

// 提供组合式函数用于Vue组件
export function useProcessorRegistry() {
  return processorRegistry;
}
```

### 第四步：第三方处理器扩展示例

创建示例展示如何开发第三方处理器扩展。

```typescript
// third-party-processors/xml-processor.ts
import type { RequestProcessor, HackbarRequest, ProcessResult, ProcessorConfig } from '@/processors/processor';

export class XmlProcessor implements RequestProcessor {
  readonly contentType = 'application/xml';
  readonly name = 'XML Processor';
  readonly version = '1.0.0';
  readonly description = '处理 application/xml 格式的请求';
  
  private config: ProcessorConfig = {
    indent: '  ',
    declaration: true
  };
  
  process(request: HackbarRequest): ProcessResult {
    try {
      const xml = this.objectToXml(request.data);
      
      return {
        body: xml,
        headers: {
          'Content-Type': this.contentType
        }
      };
    } catch (error) {
      throw new Error(`Failed to process XML data: ${error instanceof Error ? error.message : 'Unknown error'}`);
    }
  }
  
  parse(body: string): any {
    try {
      return this.xmlToObject(body);
    } catch (error) {
      throw new Error(`Failed to parse XML data: ${error instanceof Error ? error.message : 'Unknown error'}`);
    }
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
  
  serialize(data: any): string {
    return this.objectToXml(data);
  }
  
  getConfig(): ProcessorConfig {
    return { ...this.config };
  }
  
  setConfig(config: ProcessorConfig): void {
    this.config = { ...this.config, ...config };
  }
  
  private objectToXml(obj: Record<string, string>, rootName: string = 'root'): string {
    let xml = '';
    
    // 添加XML声明
    if (this.config.declaration) {
      xml += '<?xml version="1.0" encoding="UTF-8"?>\n';
    }
    
    xml += `<${rootName}>\n`;
    
    for (const [key, value] of Object.entries(obj)) {
      xml += `${this.config.indent}<${key}><![CDATA[${value}]]></${key}>\n`;
    }
    
    xml += `</${rootName}>`;
    
    return xml;
  }
  
  private xmlToObject(xml: string): Record<string, string> {
    const result: Record<string, string> = {};
    const tagRegex = /<(\w+)>(.*?)<\/\1>/gs;
    
    let match;
    while ((match = tagRegex.exec(xml)) !== null) {
      const [, tag, content] = match;
      // 处理CDATA
      const cdataMatch = content.match(/<!\[CDATA\[(.*?)\]\]>/);
      result[tag] = cdataMatch ? cdataMatch[1] : content;
    }
    
    return result;
  }
}

// 注册XML处理器
import { processorRegistry } from '@/processors';

// 在应用初始化时注册
processorRegistry.register(new XmlProcessor());
```

### 第五步：处理器管理界面

创建用户界面用于管理处理器扩展。

```vue
<!-- components/ProcessorManagement.vue -->
<template>
  <v-dialog v-model="dialog" max-width="800px" fullscreen>
    <v-card>
      <v-card-title>
        <span class="text-h5">处理器管理</span>
        <v-spacer />
        <v-btn icon @click="closeDialog">
          <v-icon>mdi-close</v-icon>
        </v-btn>
      </v-card-title>
      
      <v-card-text>
        <v-row>
          <v-col cols="12">
            <v-alert type="info" outlined>
              <div class="d-flex align-center">
                <v-icon left>mdi-information</v-icon>
                <div>
                  处理器负责处理不同内容类型的请求数据。您可以安装第三方处理器扩展来支持更多格式。
                </div>
              </div>
            </v-alert>
          </v-col>
        </v-row>
        
        <v-row>
          <v-col cols="12">
            <v-toolbar flat>
              <v-toolbar-title>已安装的处理器</v-toolbar-title>
              <v-spacer />
              <v-btn color="primary" @click="showInstallDialog = true">
                <v-icon left>mdi-plus</v-icon>
                安装处理器
              </v-btn>
            </v-toolbar>
            
            <v-data-table
              :headers="headers"
              :items="processors"
              :items-per-page="10"
              class="elevation-1"
            >
              <template #item.name="{ item }">
                <div>
                  <div class="font-weight-medium">{{ item.name }}</div>
                  <div class="text--secondary text-caption">{{ item.description }}</div>
                </div>
              </template>
              
              <template #item.version="{ item }">
                <v-chip small>
                  v{{ item.version }}
                </v-chip>
              </template>
              
              <template #item.contentType="{ item }">
                <v-chip small outlined>
                  {{ item.contentType }}
                </v-chip>
              </template>
              
              <template #item.actions="{ item }">
                <v-tooltip bottom>
                  <template #activator="{ on }">
                    <v-icon
                      small
                      class="mr-2"
                      v-on="on"
                      @click="configureProcessor(item)"
                    >
                      mdi-cog
                    </v-icon>
                  </template>
                  <span>配置</span>
                </v-tooltip>
                
                <v-tooltip bottom>
                  <template #activator="{ on }">
                    <v-icon
                      small
                      v-on="on"
                      @click="testProcessor(item)"
                    >
                      mdi-test-tube
                    </v-icon>
                  </template>
                  <span>测试</span>
                </v-tooltip>
              </template>
            </v-data-table>
          </v-col>
        </v-row>
      </v-card-text>
    </v-card>
  </v-dialog>
  
  <v-dialog v-model="showInstallDialog" max-width="600px">
    <v-card>
      <v-card-title>安装处理器</v-card-title>
      <v-card-text>
        <v-form ref="installForm">
          <v-file-input
            v-model="processorFile"
            label="选择处理器文件"
            accept=".js,.ts"
            prepend-icon="mdi-upload"
            :rules="fileRules"
          />
          
          <v-textarea
            v-model="processorCode"
            label="或粘贴处理器代码"
            rows="10"
            outlined
          />
        </v-form>
      </v-card-text>
      <v-card-actions>
        <v-spacer />
        <v-btn @click="showInstallDialog = false">取消</v-btn>
        <v-btn color="primary" @click="installProcessor">安装</v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
  
  <v-dialog v-model="showConfigDialog" max-width="600px">
    <v-card>
      <v-card-title>配置处理器</v-card-title>
      <v-card-text>
        <div class="mb-4">
          <h3>{{ selectedProcessor?.name }}</h3>
          <div class="text--secondary">{{ selectedProcessor?.description }}</div>
        </div>
        
        <v-form ref="configForm">
          <v-text-field
            v-for="(value, key) in processorConfig"
            :key="key"
            v-model="processorConfig[key]"
            :label="key"
            outlined
          />
        </v-form>
      </v-card-text>
      <v-card-actions>
        <v-spacer />
        <v-btn @click="showConfigDialog = false">取消</v-btn>
        <v-btn color="primary" @click="saveConfig">保存</v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';
import { useProcessorRegistry } from '@/processors';
import type { RequestProcessor, ProcessorMetadata } from '@/processors/processor';

const props = defineProps<{
  modelValue: boolean;
}>();

const emit = defineEmits<{
  (e: 'update:modelValue', value: boolean): void;
}>();

// 响应式数据
const dialog = ref(false);
const showInstallDialog = ref(false);
const showConfigDialog = ref(false);
const processorFile = ref<File | null>(null);
const processorCode = ref('');
const selectedProcessor = ref<RequestProcessor | null>(null);
const processorConfig = ref<Record<string, any>>({});

// 处理器注册表
const processorRegistry = useProcessorRegistry();

// 计算属性
const processors = computed<ProcessorMetadata[]>(() => {
  return processorRegistry.getAllProcessorMetadata();
});

const headers = [
  { text: '名称', value: 'name' },
  { text: '版本', value: 'version' },
  { text: '内容类型', value: 'contentType' },
  { text: '操作', value: 'actions', sortable: false }
];

const fileRules = [
  (value: File | null) => !!value || '请选择一个文件',
  (value: File | null) => !value || value.size < 1048576 || '文件大小不能超过1MB'
];

// 方法
const configureProcessor = (metadata: ProcessorMetadata) => {
  const processor = processorRegistry.getProcessor(metadata.contentType);
  if (processor) {
    selectedProcessor.value = processor;
    processorConfig.value = processor.getConfig();
    showConfigDialog.value = true;
  }
};

const saveConfig = () => {
  if (selectedProcessor.value) {
    selectedProcessor.value.setConfig(processorConfig.value);
    showConfigDialog.value = false;
  }
};

const testProcessor = (metadata: ProcessorMetadata) => {
  const processor = processorRegistry.getProcessor(metadata.contentType);
  if (processor) {
    // 执行测试逻辑
    try {
      const testData: HackbarRequest = {
        method: 'POST',
        url: 'https://example.com',
        headers: { 'Content-Type': metadata.contentType },
        body: '',
        data: { test: 'data' }
      };
      
      const result = processor.process(testData);
      alert(`处理器测试成功:\n${result.body.substring(0, 100)}...`);
    } catch (error) {
      alert(`处理器测试失败: ${error instanceof Error ? error.message : 'Unknown error'}`);
    }
  }
};

const installProcessor = async () => {
  try {
    let code = processorCode.value;
    
    // 如果选择了文件，读取文件内容
    if (processorFile.value) {
      code = await processorFile.value.text();
    }
    
    if (!code) {
      throw new Error('请提供处理器代码');
    }
    
    // 创建函数来执行代码并获取处理器
    const installFunction = new Function('processorRegistry', `
      ${code}
      // 假设代码中定义了一个 getProcessor 函数
      if (typeof getProcessor === 'function') {
        return getProcessor();
      }
      throw new Error('代码中未找到 getProcessor 函数');
    `);
    
    // 执行代码并获取处理器
    const processor = installFunction(processorRegistry);
    
    // 注册处理器
    if (processor && typeof processor === 'object' && processor.contentType) {
      processorRegistry.register(processor);
      alert('处理器安装成功');
      showInstallDialog.value = false;
      processorCode.value = '';
      processorFile.value = null;
    } else {
      throw new Error('无效的处理器对象');
    }
  } catch (error) {
    alert(`安装失败: ${error instanceof Error ? error.message : 'Unknown error'}`);
  }
};

const closeDialog = () => {
  dialog.value = false;
  emit('update:modelValue', false);
};

// 监听
watch(() => props.modelValue, (value) => {
  dialog.value = value;
});

// 生命周期
onMounted(() => {
  dialog.value = props.modelValue;
});
</script>
```

## 优势

1. **可扩展性**：可以轻松添加新的内容类型支持
2. **模块化**：每个处理器独立实现，便于维护
3. **灵活性**：运行时可以动态注册和使用处理器
4. **兼容性**：新处理器不影响现有功能
5. **可配置性**：处理器支持自定义配置
6. **可测试性**：每个处理器可以独立测试
7. **标准化**：统一的接口规范便于开发

## 最佳实践

1. **为每种内容类型实现专门的处理器**：确保每种格式都有对应的处理器
2. **提供统一的处理器注册和管理机制**：便于系统管理和使用
3. **实现处理器的验证功能**：确保输入数据的有效性
4. **提供处理器的配置选项**：允许用户自定义处理器行为
5. **编写处理器的文档和使用示例**：便于开发者理解和使用
6. **考虑处理器的性能和安全性**：优化处理效率并防止安全问题
7. **实现错误处理和恢复机制**：妥善处理处理过程中的异常
8. **支持处理器的版本管理**：便于处理器的更新和维护

## 总结

处理器扩展系统是 HackBar 架构的重要组成部分，它通过插件化设计实现了对多种内容类型的支持。通过定义标准接口、实现内置处理器、创建注册系统和支持第三方扩展，我们构建了一个灵活、可扩展的处理器框架。这一系统不仅满足了当前的处理需求，还为未来的功能扩展提供了坚实的基础。