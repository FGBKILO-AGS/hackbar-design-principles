# 高效处理 (Efficient Processing)

## 概述

HackBar 需要处理各种类型的 HTTP 请求和安全测试任务，高效的处理机制对用户体验至关重要。我们采用多种优化策略来确保系统响应迅速且资源使用合理。

## 核心思想

1. **异步处理**：使用异步操作避免阻塞主线程
2. **缓存机制**：缓存频繁使用的数据和计算结果
3. **批量操作**：合并多个操作以减少系统调用
4. **资源管理**：及时释放不需要的资源

## 代码示例

### 异步请求处理

```typescript
// background-worker/fetch-request-executor.ts
export class FetchRequestExecutor implements RequestExecutor {
  async execute(request: HackbarRequest): Promise<ExecutionResult> {
    // 使用 AbortController 实现超时控制
    const controller = new AbortController();
    const timeoutId = setTimeout(() => controller.abort(), 30000); // 30秒超时
    
    try {
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
      clearTimeout(timeoutId);
      
      if (error instanceof Error && error.name === 'AbortError') {
        return {
          success: false,
          error: '请求超时'
        };
      }
      
      return {
        success: false,
        error: error instanceof Error ? error.message : '未知错误'
      };
    }
  }
}
```

### 缓存机制

```typescript
// utils/cache.ts
export class RequestCache {
  private cache: Map<string, { data: any; timestamp: number }> = new Map();
  private readonly TTL = 5 * 60 * 1000; // 5分钟缓存时间
  
  // 生成缓存键
  private generateKey(request: HackbarRequest): string {
    return `${request.method}:${request.url}:${JSON.stringify(request.headers)}:${request.body}`;
  }
  
  // 获取缓存数据
  get(request: HackbarRequest): any | null {
    const key = this.generateKey(request);
    const cached = this.cache.get(key);
    
    if (!cached) {
      return null;
    }
    
    // 检查是否过期
    if (Date.now() - cached.timestamp > this.TTL) {
      this.cache.delete(key);
      return null;
    }
    
    return cached.data;
  }
  
  // 设置缓存数据
  set(request: HackbarRequest, data: any): void {
    const key = this.generateKey(request);
    this.cache.set(key, {
      data,
      timestamp: Date.now()
    });
    
    // 清理过期缓存
    this.cleanup();
  }
  
  // 清理过期缓存
  private cleanup(): void {
    const now = Date.now();
    for (const [key, value] of this.cache.entries()) {
      if (now - value.timestamp > this.TTL) {
        this.cache.delete(key);
      }
    }
  }
  
  // 清空缓存
  clear(): void {
    this.cache.clear();
  }
}
```

### 批量操作

```typescript
// background-worker/store.ts
export class RequestStore {
  private pendingUpdates: Array<{ id: string; data: any }> = [];
  private updateTimer: number | null = null;
  
  // 批量更新请求历史
  async batchUpdateRequests(updates: Array<{ id: string; data: any }>): Promise<void> {
    // 合并到待处理队列
    this.pendingUpdates.push(...updates);
    
    // 延迟执行更新以允许更多更新加入批次
    if (!this.updateTimer) {
      this.updateTimer = window.setTimeout(() => {
        this.executeBatchUpdate();
        this.updateTimer = null;
      }, 100); // 100ms延迟
    }
  }
  
  private async executeBatchUpdate(): Promise<void> {
    if (this.pendingUpdates.length === 0) {
      return;
    }
    
    // 创建批次更新数据
    const updates = [...this.pendingUpdates];
    this.pendingUpdates = [];
    
    try {
      // 使用事务执行批量更新
      const transaction = chrome.storage.local.transaction(['requestHistory']);
      const history = await transaction.get('requestHistory') || [];
      
      // 应用所有更新
      for (const update of updates) {
        const index = history.findIndex((req: any) => req.id === update.id);
        if (index !== -1) {
          history[index] = { ...history[index], ...update.data };
        }
      }
      
      // 保存更新后的数据
      await transaction.set({ requestHistory: history });
    } catch (error) {
      console.error('Batch update failed:', error);
      // 失败时将更新重新加入队列
      this.pendingUpdates.unshift(...updates);
    }
  }
}
```

### 资源管理

```typescript
// content-scripts/core/render.ts
export class SafeRenderer {
  private observers: MutationObserver[] = [];
  private timeouts: number[] = [];
  private intervals: number[] = [];
  
  // 安全渲染内容
  renderContent(target: HTMLElement, content: string): void {
    // 清理之前的内容和监听器
    this.cleanup();
    
    // 创建内容容器
    const container = document.createElement('div');
    container.innerHTML = content;
    
    // 添加到目标元素
    target.appendChild(container);
    
    // 监听DOM变化以清理资源
    const observer = new MutationObserver(() => {
      // 检查容器是否仍存在于DOM中
      if (!document.contains(container)) {
        this.cleanup();
      }
    });
    
    observer.observe(document.body, { childList: true, subtree: true });
    this.observers.push(observer);
  }
  
  // 设置自动清理的定时器
  setTimeout(callback: () => void, delay: number): number {
    const timeoutId = window.setTimeout(() => {
      callback();
      this.timeouts = this.timeouts.filter(id => id !== timeoutId);
    }, delay);
    
    this.timeouts.push(timeoutId);
    return timeoutId;
  }
  
  // 设置自动清理的间隔
  setInterval(callback: () => void, interval: number): number {
    const intervalId = window.setInterval(callback, interval);
    this.intervals.push(intervalId);
    return intervalId;
  }
  
  // 清理所有资源
  cleanup(): void {
    // 断开所有观察器
    for (const observer of this.observers) {
      observer.disconnect();
    }
    this.observers = [];
    
    // 清除所有定时器
    for (const timeoutId of this.timeouts) {
      window.clearTimeout(timeoutId);
    }
    this.timeouts = [];
    
    // 清除所有间隔
    for (const intervalId of this.intervals) {
      window.clearInterval(intervalId);
    }
    this.intervals = [];
  }
}
```

## 优势

1. **响应迅速**：异步处理确保界面流畅
2. **资源节约**：缓存和批量操作减少系统资源消耗
3. **用户体验**：快速响应和合理资源使用提升用户体验
4. **可扩展性**：良好的性能基础支持更多功能

## 实践建议

1. 优先使用异步操作避免阻塞主线程
2. 合理使用缓存减少重复计算
3. 合并操作减少系统调用次数
4. 及时清理不需要的资源
5. 监控性能指标并持续优化