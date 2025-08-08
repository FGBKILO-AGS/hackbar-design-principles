# 后台工作者 (Background Worker)

## 概述

HackBar 使用后台工作者来处理耗时操作，如发送 HTTP 请求、执行安全测试等。这样可以避免阻塞用户界面，提供流畅的用户体验。

## 核心思想

1. **职责分离**：将耗时操作从 UI 线程分离到后台
2. **消息传递**：通过消息机制与 UI 线程通信
3. **资源管理**：合理管理后台工作者的资源使用
4. **错误处理**：妥善处理后台操作中的错误

## 代码示例

### 后台脚本架构

```typescript
// background-worker/background.ts
// 主要的后台脚本文件
import { RequestExecutorFactory } from './request-executor';
import { RequestStore } from './store';

// 初始化组件
const requestStore = new RequestStore();
const executorFactory = new RequestExecutorFactory();

// 监听来自内容脚本或 popup 的消息
chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  // 异步处理消息
  handleMessage(message, sender)
    .then(result => sendResponse({ success: true, result }))
    .catch(error => sendResponse({ success: false, error: error.message }))
    .finally(() => {
      // 清理资源
      cleanup();
    });
  
  // 返回 true 表示异步响应
  return true;
});

async function handleMessage(message: any, sender: chrome.runtime.MessageSender): Promise<any> {
  switch (message.type) {
    case 'EXECUTE_REQUEST':
      return await executeRequest(message.payload);
      
    case 'SAVE_REQUEST':
      return await saveRequest(message.payload);
      
    case 'LOAD_REQUESTS':
      return await loadRequests();
      
    case 'DELETE_REQUEST':
      return await deleteRequest(message.payload.id);
      
    default:
      throw new Error(`Unknown message type: ${message.type}`);
  }
}

async function executeRequest(request: HackbarRequest): Promise<ExecutionResult> {
  // 根据请求类型选择执行器
  const executor = executorFactory.createExecutor(request);
  
  // 记录请求历史
  await requestStore.saveRequest(request);
  
  // 执行请求
  const result = await executor.execute(request);
  
  // 保存执行结果
  await requestStore.updateRequestResult(request.id, result);
  
  return result;
}

async function saveRequest(request: HackbarRequest): Promise<void> {
  return await requestStore.saveRequest(request);
}

async function loadRequests(): Promise<HackbarRequest[]> {
  return await requestStore.loadRequests();
}

async function deleteRequest(id: string): Promise<void> {
  return await requestStore.deleteRequest(id);
}

function cleanup(): void {
  // 清理临时资源
  executorFactory.cleanup();
  requestStore.cleanup();
}
```

### 请求执行器工厂

```typescript
// background-worker/request-executor.ts
export interface RequestExecutor {
  execute(request: HackbarRequest): Promise<ExecutionResult>;
}

export class FetchRequestExecutor implements RequestExecutor {
  async execute(request: HackbarRequest): Promise<ExecutionResult> {
    try {
      const response = await fetch(request.url, {
        method: request.method,
        headers: request.headers,
        body: request.body
      });
      
      return {
        success: true,
        status: response.status,
        headers: Object.fromEntries(response.headers.entries()),
        body: await response.text()
      };
    } catch (error) {
      return {
        success: false,
        error: error instanceof Error ? error.message : '未知错误'
      };
    }
  }
}

export class FrameRequestExecutor implements RequestExecutor {
  async execute(request: HackbarRequest): Promise<ExecutionResult> {
    // 在 iframe 中执行请求以绕过 CORS 限制
    const iframe = document.createElement('iframe');
    iframe.style.display = 'none';
    
    return new Promise((resolve) => {
      iframe.onload = async () => {
        try {
          // 在 iframe 中执行请求逻辑
          const result = await this.executeInFrame(iframe, request);
          resolve(result);
        } catch (error) {
          resolve({
            success: false,
            error: error instanceof Error ? error.message : '未知错误'
          });
        } finally {
          // 清理 iframe
          document.body.removeChild(iframe);
        }
      };
      
      document.body.appendChild(iframe);
    });
  }
  
  private async executeInFrame(iframe: HTMLIFrameElement, request: HackbarRequest): Promise<ExecutionResult> {
    // 在 iframe 中执行请求的具体实现
    // 这里省略具体实现细节
    return {
      success: true,
      status: 200,
      headers: {},
      body: 'Response from iframe'
    };
  }
}

export class RequestExecutorFactory {
  private executors: Map<string, RequestExecutor> = new Map();
  
  createExecutor(request: HackbarRequest): RequestExecutor {
    // 根据请求特征选择合适的执行器
    if (request.method === 'GET' || request.method === 'POST') {
      // 对于简单请求使用 fetch 执行器
      if (!this.executors.has('fetch')) {
        this.executors.set('fetch', new FetchRequestExecutor());
      }
      return this.executors.get('fetch')!;
    } else {
      // 对于复杂请求使用 iframe 执行器
      if (!this.executors.has('frame')) {
        this.executors.set('frame', new FrameRequestExecutor());
      }
      return this.executors.get('frame')!;
    }
  }
  
  cleanup(): void {
    this.executors.clear();
  }
}
```

### 请求存储管理

```typescript
// background-worker/store.ts
export class RequestStore {
  private readonly STORE_KEY = 'hackbar_requests';
  private cache: Map<string, HackbarRequest> = new Map();
  private cacheTime: number = 0;
  private readonly CACHE_TTL = 60 * 1000; // 1分钟缓存
  
  async saveRequest(request: HackbarRequest): Promise<void> {
    // 生成唯一ID
    if (!request.id) {
      request.id = this.generateId();
    }
    
    // 添加时间戳
    request.timestamp = Date.now();
    
    // 保存到存储
    const requests = await this.loadRequests();
    const index = requests.findIndex(r => r.id === request.id);
    
    if (index !== -1) {
      requests[index] = request;
    } else {
      requests.push(request);
    }
    
    // 保留最新的100个请求
    const sorted = requests.sort((a, b) => b.timestamp - a.timestamp);
    const latest = sorted.slice(0, 100);
    
    await chrome.storage.local.set({ [this.STORE_KEY]: latest });
    
    // 更新缓存
    this.cache.set(request.id, request);
    this.cacheTime = Date.now();
  }
  
  async loadRequests(): Promise<HackbarRequest[]> {
    // 检查缓存
    if (this.cacheTime + this.CACHE_TTL > Date.now()) {
      return Array.from(this.cache.values());
    }
    
    // 从存储加载
    const result = await chrome.storage.local.get(this.STORE_KEY);
    const requests: HackbarRequest[] = result[this.STORE_KEY] || [];
    
    // 更新缓存
    this.cache.clear();
    for (const request of requests) {
      this.cache.set(request.id, request);
    }
    this.cacheTime = Date.now();
    
    return requests;
  }
  
  async deleteRequest(id: string): Promise<void> {
    const requests = await this.loadRequests();
    const filtered = requests.filter(r => r.id !== id);
    await chrome.storage.local.set({ [this.STORE_KEY]: filtered });
    
    // 更新缓存
    this.cache.delete(id);
  }
  
  async updateRequestResult(id: string, result: ExecutionResult): Promise<void> {
    const requests = await this.loadRequests();
    const index = requests.findIndex(r => r.id === id);
    
    if (index !== -1) {
      requests[index].result = result;
      await chrome.storage.local.set({ [this.STORE_KEY]: requests });
      
      // 更新缓存
      this.cache.set(id, requests[index]);
    }
  }
  
  private generateId(): string {
    return Date.now().toString(36) + Math.random().toString(36).substr(2);
  }
  
  cleanup(): void {
    this.cache.clear();
    this.cacheTime = 0;
  }
}
```

## 优势

1. **界面流畅**：耗时操作不阻塞用户界面
2. **资源隔离**：后台操作与界面操作互不影响
3. **状态管理**：统一管理请求历史和状态
4. **错误隔离**：后台错误不会导致界面崩溃

## 实践建议

1. 将耗时操作放到后台工作者中执行
2. 使用消息机制进行前后台通信
3. 合理管理后台工作者的生命周期
4. 妥善处理后台操作中的错误
5. 优化后台工作者的资源使用