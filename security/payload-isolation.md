# 载荷隔离 (Payload Isolation)

## 概述

在 HackBar 这样的渗透测试工具中，载荷隔离是至关重要的安全措施。我们需要确保用户输入的载荷不会对扩展本身或用户的系统造成损害。

## 核心思想

1. **沙箱执行**：在受限环境中执行用户载荷
2. **输入验证**：严格验证所有用户输入
3. **权限控制**：最小权限原则，只授予必要权限
4. **数据隔离**：隔离用户数据和扩展核心数据

## 代码示例

### 内容脚本隔离

```typescript
// content-scripts/core/post.ts
// 内容脚本在页面上下文中执行，与扩展隔离
export function executePostRequest(formData: FormData) {
  // 创建一个表单元素
  const form = document.createElement('form');
  form.method = 'POST';
  form.style.display = 'none';
  
  // 添加数据字段
  for (const [key, value] of Object.entries(formData)) {
    const input = document.createElement('input');
    input.type = 'hidden';
    input.name = key;
    input.value = value as string;
    form.appendChild(input);
  }
  
  // 提交表单
  document.body.appendChild(form);
  form.submit();
  
  // 清理
  document.body.removeChild(form);
}
```

### 背景脚本权限控制

```typescript
// background-worker/request-executor.ts
export class RequestExecutor {
  // 限制可以设置的请求头
  private allowedHeaders = [
    'Content-Type',
    'User-Agent',
    'Accept',
    'Accept-Language',
    'Accept-Encoding'
  ];
  
  async executeRequest(request: HackbarRequest): Promise<Response> {
    // 验证请求URL
    if (!this.isValidUrl(request.url)) {
      throw new Error('Invalid URL');
    }
    
    // 过滤请求头
    const safeHeaders: Record<string, string> = {};
    for (const [key, value] of Object.entries(request.headers)) {
      if (this.allowedHeaders.includes(key)) {
        safeHeaders[key] = value;
      }
    }
    
    // 使用浏览器API发送请求
    return fetch(request.url, {
      method: request.method,
      headers: safeHeaders,
      body: request.body
    });
  }
  
  private isValidUrl(url: string): boolean {
    try {
      const parsedUrl = new URL(url);
      return ['http:', 'https:'].includes(parsedUrl.protocol);
    } catch {
      return false;
    }
  }
}
```

### 自定义载荷存储隔离

```typescript
// stores/custom-payload.ts
import { defineStore } from 'pinia';

export const useCustomPayloadStore = defineStore('customPayload', () => {
  // 使用浏览器存储API存储载荷
  const payloads = ref<CustomPayload[]>([]);
  
  // 从存储中加载载荷
  const loadPayloads = async () => {
    try {
      const stored = await chrome.storage.local.get(['customPayloads']);
      payloads.value = stored.customPayloads || [];
    } catch (error) {
      console.error('Failed to load payloads:', error);
      payloads.value = [];
    }
  };
  
  // 保存载荷到存储
  const savePayloads = async () => {
    try {
      // 验证载荷内容
      const validatedPayloads = payloads.value.map(payload => ({
        ...payload,
        // 确保只保存预期的字段
        id: typeof payload.id === 'string' ? payload.id : '',
        name: typeof payload.name === 'string' ? payload.name : '',
        content: typeof payload.content === 'string' ? payload.content : '',
        category: typeof payload.category === 'string' ? payload.category : 'general'
      }));
      
      await chrome.storage.local.set({ 
        customPayloads: validatedPayloads 
      });
    } catch (error) {
      console.error('Failed to save payloads:', error);
    }
  };
  
  // 添加新载荷
  const addPayload = (payload: Omit<CustomPayload, 'id'>) => {
    const newPayload: CustomPayload = {
      id: crypto.randomUUID(),
      ...payload
    };
    
    // 验证载荷内容
    if (!newPayload.name || !newPayload.content) {
      throw new Error('Payload name and content are required');
    }
    
    payloads.value.push(newPayload);
    return savePayloads();
  };
  
  return {
    payloads,
    loadPayloads,
    addPayload
  };
});
```

### 载荷执行安全检查

```typescript
// utils/functions.ts
export class PayloadSecurity {
  // 检查载荷是否包含危险代码
  static isSafePayload(payload: string): boolean {
    const dangerousPatterns = [
      /document\.cookie/i,
      /localStorage/i,
      /sessionStorage/i,
      /eval\s*\(/i,
      /Function\s*\(/i,
      /setTimeout\s*\([^,]+,\s*['"`\d]/i,
      /setInterval\s*\([^,]+,\s*['"`\d]/i
    ];
    
    return !dangerousPatterns.some(pattern => pattern.test(payload));
  }
  
  // 转义HTML特殊字符
  static escapeHtml(text: string): string {
    const map: Record<string, string> = {
      '&': '&amp;',
      '<': '&lt;',
      '>': '&gt;',
      '"': '&quot;',
      "'": '&#039;'
    };
    
    return text.replace(/[&<>"']/g, (m) => map[m]);
  }
  
  // 安全执行载荷预览
  static previewPayload(payload: string): string {
    if (!this.isSafePayload(payload)) {
      throw new Error('Potentially unsafe payload detected');
    }
    
    // 返回转义后的内容用于预览
    return this.escapeHtml(payload);
  }
}
```

## 优势

1. **防止XSS攻击**：隔离用户输入，防止跨站脚本攻击
2. **保护用户数据**：避免载荷访问敏感用户信息
3. **系统安全**：防止恶意载荷损害用户系统
4. **合规性**：符合浏览器扩展安全规范

## 实践建议

1. 始终验证和清理用户输入
2. 使用内容脚本在页面上下文中执行操作
3. 限制可设置的HTTP头
4. 对敏感操作进行额外的安全检查
5. 定期审查安全措施的有效性