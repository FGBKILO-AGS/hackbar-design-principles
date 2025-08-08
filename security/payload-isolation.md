# 载荷隔离 (Payload Isolation)

## 原始定义

载荷隔离是一种安全机制，用于将用户提供的内容与系统核心功能分离，以防止恶意载荷对系统本身或用户环境造成损害。这种隔离通过沙箱执行、输入验证和权限控制等方式实现。

## 详细解释

在渗透测试工具如 HackBar 中，载荷隔离是至关重要的安全措施。我们需要确保用户输入的载荷不会对扩展本身或用户的系统造成损害。载荷隔离通过多种技术手段实现，包括沙箱执行、输入验证、权限控制和数据隔离。

### 核心概念

1. **沙箱执行**：在受限环境中执行用户载荷
2. **输入验证**：严格验证所有用户输入
3. **权限控制**：最小权限原则，只授予必要权限
4. **数据隔离**：隔离用户数据和扩展核心数据

### 在 HackBar 中的应用

在 HackBar 这样的渗透测试工具中，载荷隔离是至关重要的安全措施。我们需要确保用户输入的载荷不会对扩展本身或用户的系统造成损害。

## 实施步骤

### 第一步：内容脚本隔离

在内容脚本中执行用户载荷，与扩展核心隔离。

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
  
  // 添加安全检查
  if (!isSafeFormData(formData)) {
    throw new Error('Unsafe form data detected');
  }
  
  // 提交表单
  document.body.appendChild(form);
  form.submit();
  
  // 清理
  setTimeout(() => {
    document.body.removeChild(form);
  }, 1000);
}

// 安全检查函数
function isSafeFormData(formData: FormData): boolean {
  // 检查字段名是否安全
  for (const key in formData) {
    if (!/^[a-zA-Z0-9_-]+$/.test(key)) {
      return false;
    }
  }
  
  // 检查字段值是否包含危险内容
  for (const value of Object.values(formData)) {
    if (typeof value === 'string' && containsDangerousContent(value)) {
      return false;
    }
  }
  
  return true;
}

// 检查是否包含危险内容
function containsDangerousContent(value: string): boolean {
  const dangerousPatterns = [
    /<script/i,
    /javascript:/i,
    /on\w+\s*=/i,
    /eval\s*\(/i,
    /document\.cookie/i
  ];
  
  return dangerousPatterns.some(pattern => pattern.test(value));
}
```

### 第二步：背景脚本权限控制

在背景脚本中实施严格的权限控制。

```typescript
// background-worker/request-executor.ts
export class RequestExecutor {
  // 限制可以设置的请求头
  private allowedHeaders = [
    'Content-Type',
    'User-Agent',
    'Accept',
    'Accept-Language',
    'Accept-Encoding',
    'Cache-Control',
    'Referer'
  ];
  
  async executeRequest(request: HackbarRequest): Promise<Response> {
    // 验证请求URL
    if (!this.isValidUrl(request.url)) {
      throw new Error('Invalid URL');
    }
    
    // 验证请求方法
    if (!this.isValidMethod(request.method)) {
      throw new Error('Invalid HTTP method');
    }
    
    // 过滤请求头
    const safeHeaders: Record<string, string> = {};
    for (const [key, value] of Object.entries(request.headers)) {
      if (this.allowedHeaders.includes(key)) {
        // 验证头值安全性
        if (this.isSafeHeaderValue(value)) {
          safeHeaders[key] = value;
        }
      }
    }
    
    // 验证请求体
    if (!this.isSafeRequestBody(request.body)) {
      throw new Error('Unsafe request body');
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
  
  private isValidMethod(method: string): boolean {
    const allowedMethods = ['GET', 'POST', 'PUT', 'DELETE', 'PATCH', 'HEAD', 'OPTIONS'];
    return allowedMethods.includes(method.toUpperCase());
  }
  
  private isSafeHeaderValue(value: string): boolean {
    // 检查头值是否包含控制字符或换行符
    return !/[\x00-\x1f\x7f]|[\r\n]/.test(value);
  }
  
  private isSafeRequestBody(body: string): boolean {
    // 对于大载荷，只检查前1000个字符
    const checkContent = body.length > 1000 ? body.substring(0, 1000) : body;
    
    // 检查是否包含明显的恶意内容
    const dangerousPatterns = [
      /<script/i,
      /javascript:/i,
      /data:text\/html/i
    ];
    
    return !dangerousPatterns.some(pattern => pattern.test(checkContent));
  }
}
```

### 第三步：自定义载荷存储隔离

实现安全的自定义载荷存储机制。

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
      
      // 验证载荷数据安全性
      payloads.value = payloads.value.filter(payload => isValidPayload(payload));
    } catch (error) {
      console.error('Failed to load payloads:', error);
      payloads.value = [];
    }
  };
  
  // 保存载荷到存储
  const savePayloads = async () => {
    try {
      // 验证载荷内容
      const validatedPayloads = payloads.value
        .filter(payload => isValidPayload(payload))
        .map(payload => sanitizePayload(payload));
      
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
      ...payload,
      createdAt: Date.now(),
      updatedAt: Date.now()
    };
    
    // 验证载荷内容
    if (!isValidPayload(newPayload)) {
      throw new Error('Invalid payload content');
    }
    
    // 清理载荷内容
    const sanitizedPayload = sanitizePayload(newPayload);
    
    payloads.value.push(sanitizedPayload);
    return savePayloads();
  };
  
  // 验证载荷有效性
  function isValidPayload(payload: CustomPayload): boolean {
    // 检查必需字段
    if (!payload.name || !payload.content) {
      return false;
    }
    
    // 检查字段长度
    if (payload.name.length > 100 || payload.content.length > 10000) {
      return false;
    }
    
    // 检查字段内容安全性
    if (containsDangerousContent(payload.name) || 
        containsDangerousContent(payload.content)) {
      return false;
    }
    
    // 检查分类有效性
    const validCategories = ['general', 'xss', 'sqli', 'lfi', 'ssrf', 'ssti'];
    if (payload.category && !validCategories.includes(payload.category)) {
      return false;
    }
    
    return true;
  }
  
  // 清理载荷内容
  function sanitizePayload(payload: CustomPayload): CustomPayload {
    return {
      ...payload,
      name: sanitizeString(payload.name),
      content: sanitizeString(payload.content),
      category: payload.category || 'general'
    };
  }
  
  // 字符串清理函数
  function sanitizeString(str: string): string {
    // 移除控制字符
    return str.replace(/[\x00-\x08\x0B\x0C\x0E-\x1F\x7F]/g, '');
  }
  
  // 检查是否包含危险内容
  function containsDangerousContent(str: string): boolean {
    const dangerousPatterns = [
      /<script/i,
      /javascript:/i,
      /data:text\/html/i,
      /vbscript:/i,
      /on\w+\s*=/i
    ];
    
    return dangerousPatterns.some(pattern => pattern.test(str));
  }
  
  return {
    payloads,
    loadPayloads,
    addPayload
  };
});
```

### 第四步：载荷执行安全检查

实现载荷执行前的安全检查机制。

```typescript
// utils/functions.ts
export class PayloadSecurity {
  // 检查载荷是否包含危险代码
  static isSafePayload(payload: string): boolean {
    // 长度检查
    if (payload.length > 100000) {
      return false;
    }
    
    const dangerousPatterns = [
      /document\.cookie/i,
      /localStorage/i,
      /sessionStorage/i,
      /eval\s*\(/i,
      /Function\s*\(/i,
      /setTimeout\s*\([^,]+,\s*['"`\d]/i,
      /setInterval\s*\([^,]+,\s*['"`\d]/i,
      /import\s*\(/i,
      /fetch\s*\(/i,
      /XMLHttpRequest/i
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
  
  // 安全解析载荷
  static parsePayload(payload: string): any {
    if (!this.isSafePayload(payload)) {
      throw new Error('Potentially unsafe payload detected');
    }
    
    try {
      // 尝试解析为JSON
      return JSON.parse(payload);
    } catch {
      // 如果不是JSON，返回原始字符串
      return payload;
    }
  }
  
  // 创建安全的执行环境
  static createSandboxExecutor(): (code: string) => any {
    // 在受限环境中创建执行器
    return function(code: string) {
      if (!PayloadSecurity.isSafePayload(code)) {
        throw new Error('Unsafe code detected');
      }
      
      // 使用 Function 构造器而不是 eval
      try {
        const func = new Function('return (' + code + ')');
        return func();
      } catch (error) {
        console.error('Failed to execute code:', error);
        throw error;
      }
    };
  }
}
```

## 优势

1. **防止XSS攻击**：隔离用户输入，防止跨站脚本攻击
2. **保护用户数据**：避免载荷访问敏感用户信息
3. **系统安全**：防止恶意载荷损害用户系统
4. **合规性**：符合浏览器扩展安全规范
5. **可控性**：对载荷执行进行细粒度控制

## 最佳实践

1. **始终验证和清理用户输入**：对所有用户输入进行严格验证
2. **使用内容脚本在页面上下文中执行操作**：将潜在危险操作与扩展核心隔离
3. **限制可设置的HTTP头**：只允许安全的HTTP头字段
4. **对敏感操作进行额外的安全检查**：在关键操作前增加多重验证
5. **定期审查安全措施的有效性**：持续更新安全策略以应对新威胁
6. **实施最小权限原则**：只请求完成功能所必需的权限
7. **使用安全的存储机制**：对存储的数据进行加密和验证

## 总结

载荷隔离是 HackBar 安全架构的核心组成部分。通过在内容脚本中执行用户载荷、在背景脚本中实施权限控制、安全存储自定义载荷以及执行前的安全检查，我们能够有效防止恶意载荷对系统和用户造成损害。这种多层防护机制确保了 HackBar 作为一个渗透测试工具的安全性和可靠性。