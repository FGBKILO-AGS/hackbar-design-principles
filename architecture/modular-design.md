# 模块化设计原则 (Modular Design Principle)

## 概述

HackBar 采用了模块化设计，将不同功能划分为独立的模块，每个模块都有明确的职责和接口。这种设计使得代码更易于维护、测试和扩展。

## 核心思想

1. **单一职责原则**：每个模块只负责一个特定的功能
2. **松耦合**：模块之间通过明确定义的接口进行通信
3. **高内聚**：模块内部的组件紧密相关，共同完成特定任务

## 代码示例

在 HackBar 中，模块化设计体现在多个方面：

### 目录结构

```
src/
├── background-worker/     # 后台工作者模块
├── components/            # Vue 组件模块
├── content-scripts/       # 内容脚本模块
├── generators/            # 生成器模块
├── plugins/               # 插件模块
├── processors/            # 请求处理器模块
├── stores/                # 状态管理模块
└── utils/                 # 工具函数模块
```

### 处理器模块示例

```typescript
// processors/processor.ts - 基础处理器接口
export interface RequestProcessor {
  readonly contentType: string;
  process(request: HackbarRequest): ProcessResult;
  parse(body: string): any;
}

// processors/implementations/1-application-x-www-form-urlencoded.ts - 具体实现
export class FormUrlencodedProcessor implements RequestProcessor {
  readonly contentType = 'application/x-www-form-urlencoded';
  
  process(request: HackbarRequest): ProcessResult {
    // 实现处理逻辑
    return {
      body: this.encodeFormData(request.data),
      headers: { 'Content-Type': this.contentType }
    };
  }
  
  parse(body: string): any {
    // 实现解析逻辑
    return Object.fromEntries(new URLSearchParams(body));
  }
  
  private encodeFormData(data: Record<string, string>): string {
    const params = new URLSearchParams();
    for (const [key, value] of Object.entries(data)) {
      params.append(key, value);
    }
    return params.toString();
  }
}
```

### 插件模块示例

```typescript
// plugins/pinia.ts - 状态管理插件
import { createPinia } from 'pinia';
import type { App } from 'vue';

export default function installPinia(app: App) {
  const pinia = createPinia();
  app.use(pinia);
  return pinia;
}

// plugins/vuetify.ts - UI框架插件
import 'vuetify/styles';
import { createVuetify } from 'vuetify';
import type { App } from 'vue';

export default function installVuetify(app: App) {
  const vuetify = createVuetify({
    // Vuetify 配置
  });
  app.use(vuetify);
  return vuetify;
}
```

## 优势

1. **易于维护**：修改某个功能只需要关注对应的模块
2. **易于测试**：每个模块可以独立进行单元测试
3. **易于扩展**：添加新功能只需要创建新的模块
4. **团队协作**：不同开发者可以并行开发不同模块

## 实践建议

1. 在设计新功能时，首先考虑如何将其划分为独立模块
2. 定义清晰的模块接口，避免模块间直接依赖具体实现
3. 保持模块的独立性，避免出现循环依赖
4. 为每个模块编写清晰的文档和示例