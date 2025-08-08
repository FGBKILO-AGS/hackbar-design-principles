# 后台工作者 (Background Worker)

## 原始定义

后台工作者是一种在应用程序主执行线程之外运行的独立执行单元，用于处理耗时操作而不阻塞用户界面。在浏览器扩展中，后台工作者通常运行在后台脚本中，负责处理不需要用户交互的长时间运行任务。

## 详细解释

HackBar 使用后台工作者来处理耗时操作，如发送 HTTP 请求、执行安全测试等。这样可以避免阻塞用户界面，提供流畅的用户体验。后台工作者是浏览器扩展架构中的重要组成部分，它们在独立的执行环境中运行，与内容脚本和弹出窗口等其他组件通过消息传递进行通信。

### 核心概念

1. **职责分离**：将耗时操作从 UI 线程分离到后台
2. **消息传递**：通过消息机制与 UI 线程通信
3. **资源管理**：合理管理后台工作者的资源使用
4. **错误处理**：妥善处理后台操作中的错误
5. **生命周期管理**：管理后台工作者的启动、运行和停止

### 在 HackBar 中的应用

HackBar 使用后台工作者来处理耗时操作，如发送 HTTP 请求、执行安全测试等。这样可以避免阻塞用户界面，提供流畅的用户体验。

## 实施步骤

### 第一步：后台脚本架构设计

设计后台脚本的整体架构，包括消息处理和组件初始化。

```typescript
// background-worker/background.ts
// 主要的后台脚本文件
import { RequestExecutorFactory } from './request-executor';
import { RequestStore } from './store';
import { performanceMonitor } from '@/utils/performance-monitor';

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

// 监听连接事件
chrome.runtime.onConnect.addListener((port) => {
  console.log('Background worker connected to port:', port.name);
  
  port.onMessage.addListener((message) => {
    handlePortMessage(message, port);
  });
  
  port.onDisconnect.addListener(() => {
    console.log('Port disconnected:', port.name);
  });
});

async function handleMessage(message: any, sender: chrome.runtime.MessageSender): Promise<any> {
  return performanceMonitor.timeAsync(`handleMessage:${message.type}`, async () => {
    switch (message.type) {
      case 'EXECUTE_REQUEST':
        return await executeRequest(message.payload);
        
      case 'SAVE_REQUEST':
        return await saveRequest(message.payload);
        
      case 'LOAD_REQUESTS':
        return await loadRequests();
        
      case 'DELETE_REQUEST':
        return await deleteRequest(message.payload.id);
        
      case 'BATCH_EXECUTE':
        return await batchExecuteRequests(message.payload);
        
      default:
        throw new Error(`Unknown message type: ${message.type}`);
    }
  });
}

async function handlePortMessage(message: any, port: chrome.runtime.Port): Promise<void> {
  try {
    switch (message.type) {
      case 'START_STREAMING':
        // 处理流式数据传输
        await handleStreamingRequest(message.payload, port);
        break;
        
      case 'PROGRESS_UPDATE':
        // 处理进度更新
        port.postMessage({ type: 'PROGRESS_UPDATE', payload: message.payload });
        break;
        
      default:
        port.postMessage({ type: 'ERROR', error: `Unknown message type: ${message.type}` });
    }
  } catch (error) {
    port.postMessage({ 
      type: 'ERROR', 
      error: error instanceof Error ? error.message : 'Unknown error' 
    });
  }
}

async function executeRequest(request: HackbarRequest): Promise<ExecutionResult> {
  return performanceMonitor.timeAsync('executeRequest', async () => {
    // 根据请求类型选择执行器
    const executor = executorFactory.createExecutor(request);
    
    // 记录请求历史
    await requestStore.saveRequest(request);
    
    // 执行请求
    const result = await executor.execute(request);
    
    // 保存执行结果
    await requestStore.updateRequestResult(request.id, result);
    
    return result;
  });
}

async function batchExecuteRequests(requests: HackbarRequest[]): Promise<ExecutionResult[]> {
  return performanceMonitor.timeAsync('batchExecuteRequests', async () => {
    const executor = executorFactory.createExecutor(requests[0]);
    return await executor.executeBatch(requests);
  });
}

async function saveRequest(request: HackbarRequest): Promise<void> {
  return performanceMonitor.timeAsync('saveRequest', async () => {
    return await requestStore.saveRequest(request);
  });
}

async function loadRequests(): Promise<HackbarRequest[]> {
  return performanceMonitor.timeAsync('loadRequests', async () => {
    return await requestStore.loadRequests();
  });
}

async function deleteRequest(id: string): Promise<void> {
  return performanceMonitor.timeAsync('deleteRequest', async () => {
    return await requestStore.deleteRequest(id);
  });
}

async function handleStreamingRequest(payload: any, port: chrome.runtime.Port): Promise<void> {
  // 实现流式请求处理
  try {
    const request = payload.request;
    const executor = executorFactory.createExecutor(request);
    
    // 发送开始消息
    port.postMessage({ type: 'STREAM_START' });
    
    // 执行流式请求
    await executor.executeStreaming(request, (chunk) => {
      port.postMessage({ type: 'STREAM_DATA', payload: chunk });
    });
    
    // 发送结束消息
    port.postMessage({ type: 'STREAM_END' });
  } catch (error) {
    port.postMessage({ 
      type: 'STREAM_ERROR', 
      error: error instanceof Error ? error.message : 'Unknown error' 
    });
  }
}

function cleanup(): void {
  // 清理临时资源
  executorFactory.cleanup();
  requestStore.cleanup();
}
```

### 第二步：请求执行器工厂实现

实现请求执行器工厂，用于创建不同类型的请求执行器。

```typescript
// background-worker/request-executor.ts
export interface RequestExecutor {
  execute(request: HackbarRequest): Promise<ExecutionResult>;
  executeBatch?(requests: HackbarRequest[]): Promise<ExecutionResult[]>;
  executeStreaming?(request: HackbarRequest, onData: (chunk: string) => void): Promise<void>;
}

export class FetchRequestExecutor implements RequestExecutor {
  async execute(request: HackbarRequest): Promise<ExecutionResult> {
    try {
      const controller = new AbortController();
      const timeoutId = setTimeout(() => controller.abort(), 30000);
      
      const response = await fetch(request.url, {
        method: request.method,
        headers: request.headers,
        body: request.body,
        signal: controller.signal
      });
      
      clearTimeout(timeoutId);
      
      return {
        success: true,
        status: response.status,
        headers: Object.fromEntries(response.headers.entries()),
        body: await response.text()
      };
    } catch (error) {
      if (error instanceof Error && error.name === 'AbortError') {
        return {
          success: false,
          error: 'Request timeout'
        };
      }
      
      return {
        success: false,
        error: error instanceof Error ? error.message : 'Unknown error'
      };
    }
  }
  
  async executeBatch(requests: HackbarRequest[]): Promise<ExecutionResult[]> {
    // 限制并发数量
    const concurrencyLimit = 5;
    const results: ExecutionResult[] = [];
    
    for (let i = 0; i < requests.length; i += concurrencyLimit) {
      const batch = requests.slice(i, i + concurrencyLimit);
      const batchPromises = batch.map(req => this.execute(req));
      const batchResults = await Promise.all(batchPromises);
      results.push(...batchResults);
    }
    
    return results;
  }
  
  async executeStreaming(request: HackbarRequest, onData: (chunk: string) => void): Promise<void> {
    try {
      const response = await fetch(request.url, {
        method: request.method,
        headers: request.headers,
        body: request.body
      });
      
      if (!response.body) {
        throw new Error('Response body is not readable');
      }
      
      const reader = response.body.getReader();
      const decoder = new TextDecoder();
      
      try {
        while (true) {
          const { done, value } = await reader.read();
          if (done) break;
          
          const chunk = decoder.decode(value, { stream: true });
          onData(chunk);
        }
      } finally {
        reader.releaseLock();
      }
    } catch (error) {
      throw new Error(`Streaming error: ${error instanceof Error ? error.message : 'Unknown error'}`);
    }
  }
}

export class FrameRequestExecutor implements RequestExecutor {
  async execute(request: HackbarRequest): Promise<ExecutionResult> {
    // 在 iframe 中执行请求以绕过 CORS 限制
    return new Promise((resolve) => {
      const iframe = document.createElement('iframe');
      iframe.style.display = 'none';
      
      iframe.onload = async () => {
        try {
          // 在 iframe 中执行请求逻辑
          const result = await this.executeInFrame(iframe, request);
          resolve(result);
        } catch (error) {
          resolve({
            success: false,
            error: error instanceof Error ? error.message : 'Unknown error'
          });
        } finally {
          // 清理 iframe
          document.body.removeChild(iframe);
        }
      };
      
      iframe.onerror = () => {
        resolve({
          success: false,
          error: 'Failed to load iframe'
        });
        document.body.removeChild(iframe);
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

### 第三步：请求存储管理实现

实现请求存储管理，用于持久化请求数据。

```typescript
// background-worker/store.ts
export class RequestStore {
  private readonly STORE_KEY = 'hackbar_requests';
  private cache: Map<string, HackbarRequest> = new Map();
  private cacheTime: number = 0;
  private readonly CACHE_TTL = 60 * 1000; // 1分钟缓存
  private readonly MAX_HISTORY_SIZE = 1000; // 最大历史记录数
  
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
    
    // 保留最新的请求历史
    const sorted = requests.sort((a, b) => (b.timestamp || 0) - (a.timestamp || 0));
    const latest = sorted.slice(0, this.MAX_HISTORY_SIZE);
    
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
  
  // 获取存储统计信息
  async getStats(): Promise<{ 
    totalRequests: number; 
    storageSize: number; 
    cacheSize: number 
  }> {
    const requests = await this.loadRequests();
    const storageData = await chrome.storage.local.get();
    const storageSize = JSON.stringify(storageData).length;
    
    return {
      totalRequests: requests.length,
      storageSize,
      cacheSize: this.cache.size
    };
  }
}
```

### 第四步：后台工作者生命周期管理

实现后台工作者的生命周期管理机制。

```typescript
// background-worker/lifecycle.ts
export class BackgroundWorkerLifecycle {
  private isInitialized = false;
  private isShuttingDown = false;
  private components: Array<{ name: string; cleanup: () => Promise<void> }> = [];
  
  async initialize(): Promise<void> {
    if (this.isInitialized) {
      return;
    }
    
    try {
      console.log('Initializing background worker...');
      
      // 初始化各个组件
      await this.initializeComponents();
      
      // 注册清理处理程序
      this.registerCleanupHandlers();
      
      this.isInitialized = true;
      console.log('Background worker initialized successfully');
    } catch (error) {
      console.error('Failed to initialize background worker:', error);
      throw error;
    }
  }
  
  private async initializeComponents(): Promise<void> {
    // 这里可以初始化各种组件
    // 例如：数据库连接、网络监听器、定时任务等
    console.log('Initializing components...');
  }
  
  private registerCleanupHandlers(): void {
    // 监听扩展卸载事件
    chrome.runtime.onSuspend.addListener(() => {
      this.shutdown();
    });
    
    // 监听系统关机事件
    if (chrome.runtime.onShutdown) {
      chrome.runtime.onShutdown.addListener(() => {
        this.shutdown();
      });
    }
  }
  
  async shutdown(): Promise<void> {
    if (this.isShuttingDown) {
      return;
    }
    
    this.isShuttingDown = true;
    console.log('Shutting down background worker...');
    
    try {
      // 清理所有组件
      await this.cleanupComponents();
      
      console.log('Background worker shut down successfully');
    } catch (error) {
      console.error('Error during shutdown:', error);
    }
  }
  
  private async cleanupComponents(): Promise<void> {
    // 按相反顺序清理组件
    for (let i = this.components.length - 1; i >= 0; i--) {
      const component = this.components[i];
      try {
        await component.cleanup();
        console.log(`Cleaned up component: ${component.name}`);
      } catch (error) {
        console.error(`Failed to clean up component ${component.name}:`, error);
      }
    }
  }
  
  // 注册需要清理的组件
  registerComponent(name: string, cleanup: () => Promise<void>): void {
    this.components.push({ name, cleanup });
  }
  
  // 检查是否已初始化
  isReady(): boolean {
    return this.isInitialized && !this.isShuttingDown;
  }
}

// 创建全局生命周期管理器
export const backgroundLifecycle = new BackgroundWorkerLifecycle();

// 在后台脚本启动时初始化
backgroundLifecycle.initialize().catch(error => {
  console.error('Failed to initialize background worker lifecycle:', error);
});
```

## 优势

1. **界面流畅**：耗时操作不阻塞用户界面
2. **资源隔离**：后台操作与界面操作互不影响
3. **状态管理**：统一管理请求历史和状态
4. **错误隔离**：后台错误不会导致界面崩溃
5. **可扩展性**：支持复杂的后台任务处理
6. **通信机制**：通过消息传递实现组件间通信

## 最佳实践

1. **将耗时操作放到后台工作者中执行**：确保用户界面响应性
2. **使用消息机制进行前后台通信**：通过标准化的消息格式进行通信
3. **合理管理后台工作者的生命周期**：正确初始化和清理资源
4. **妥善处理后台操作中的错误**：实现错误恢复和报告机制
5. **优化后台工作者的资源使用**：避免内存泄漏和资源浪费
6. **实施适当的超时机制**：避免长时间运行的任务影响系统性能
7. **监控后台工作者性能**：使用性能监控工具识别和解决性能问题

## 总结

后台工作者是 HackBar 架构中的关键组件，负责处理耗时操作以保持用户界面的响应性。通过合理的架构设计、组件管理和生命周期控制，我们能够构建一个高效、可靠的后台处理系统。后台工作者与用户界面组件通过消息传递进行通信，实现了职责分离和系统解耦，为扩展提供了良好的可扩展性和维护性。