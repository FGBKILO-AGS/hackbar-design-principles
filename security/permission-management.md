# 权限管理 (Permission Management)

## 概述

浏览器扩展的权限管理是确保用户隐私和安全的重要机制。HackBar 遵循最小权限原则，只请求完成其功能所必需的权限，并向用户清楚地解释每个权限的用途。

## 核心思想

1. **最小权限原则**：只请求完成功能所必需的权限
2. **透明性**：向用户清楚解释每个权限的用途
3. **运行时请求**：在需要时才请求权限，而不是一次性请求所有权限
4. **权限撤销处理**：优雅处理权限被撤销的情况

## 代码示例

### Manifest 权限声明

```json
// manifest.json (Chrome)
{
  "name": "HackBar",
  "permissions": [
    "storage",
    "scripting",
    "webRequest",
    "declarativeNetRequest"
  ],
  "host_permissions": [
    "<all_urls>"
  ],
  "optional_permissions": [
    "clipboardWrite"
  ]
}
```

```json
// manifest.json (Firefox)
{
  "name": "HackBar",
  "permissions": [
    "storage",
    "scripting",
    "webRequest",
    "webRequestBlocking",
    "<all_urls>"
  ],
  "optional_permissions": [
    "clipboardWrite"
  ]
}
```

### 权限使用说明

```typescript
// permission.ts
export class PermissionManager {
  // 检查是否拥有特定权限
  static async hasPermission(permission: string): Promise<boolean> {
    try {
      return await chrome.permissions.contains({
        permissions: [permission]
      });
    } catch (error) {
      console.error(`Error checking permission ${permission}:`, error);
      return false;
    }
  }
  
  // 请求权限
  static async requestPermission(permission: string): Promise<boolean> {
    try {
      return await chrome.permissions.request({
        permissions: [permission]
      });
    } catch (error) {
      console.error(`Error requesting permission ${permission}:`, error);
      return false;
    }
  }
  
  // 解释权限用途
  static getPermissionDescription(permission: string): string {
    const descriptions: Record<string, string> = {
      'storage': '保存主题偏好设置和自定义载荷',
      'scripting': '执行POST请求和测试功能',
      'webRequest': '记录请求信息并在完成后清理',
      'declarativeNetRequest': '根据设置修改HTTP头值',
      'clipboardWrite': '复制载荷到剪贴板'
    };
    
    return descriptions[permission] || '此权限用于扩展功能';
  }
}
```

### 权限检查和请求

```vue
<!-- PermissionRequest.vue -->
<template>
  <v-dialog v-model="showDialog" max-width="500px">
    <v-card>
      <v-card-title>需要权限</v-card-title>
      <v-card-text>
        <p>此功能需要以下权限：</p>
        <v-list>
          <v-list-item v-for="perm in requiredPermissions" :key="perm">
            <v-list-item-content>
              <v-list-item-title>{{ perm }}</v-list-item-title>
              <v-list-item-subtitle>
                {{ getPermissionDescription(perm) }}
              </v-list-item-subtitle>
            </v-list-item-content>
          </v-list-item>
        </v-list>
      </v-card-text>
      <v-card-actions>
        <v-btn @click="cancel">取消</v-btn>
        <v-btn color="primary" @click="grantPermission">授予权限</v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import { PermissionManager } from '@/utils/permissions';

const showDialog = defineModel<boolean>('show', { required: true });
const requiredPermissions = defineModel<string[]>('permissions', { required: true });

const emit = defineEmits<{
  (e: 'granted'): void;
  (e: 'denied'): void;
}>();

const getPermissionDescription = (permission: string): string => {
  return PermissionManager.getPermissionDescription(permission);
};

const grantPermission = async () => {
  try {
    const granted = await PermissionManager.requestPermission(requiredPermissions.value);
    if (granted) {
      emit('granted');
    } else {
      emit('denied');
    }
    showDialog.value = false;
  } catch (error) {
    console.error('Failed to request permission:', error);
    emit('denied');
    showDialog.value = false;
  }
};

const cancel = () => {
  emit('denied');
  showDialog.value = false;
};
</script>
```

### 功能中的权限检查

```typescript
// background-worker/fetch-request-executor.ts
export class FetchRequestExecutor {
  async execute(request: HackbarRequest): Promise<ExecutionResult> {
    // 检查网络请求权限
    const hasWebRequestPermission = await PermissionManager.hasPermission('webRequest');
    if (!hasWebRequestPermission) {
      throw new Error('缺少网络请求权限');
    }
    
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
```

### 权限变更监听

```typescript
// background-worker/background.ts
chrome.permissions.onAdded.addListener((permissions) => {
  console.log('Permissions added:', permissions);
  // 处理新增权限
});

chrome.permissions.onRemoved.addListener((permissions) => {
  console.log('Permissions removed:', permissions);
  // 处理移除的权限，禁用相关功能
});

// 在应用启动时检查权限
async function initializeApp() {
  const permissions = await chrome.permissions.getAll();
  console.log('Current permissions:', permissions);
  
  // 根据权限启用/禁用功能
  updateFeatureAvailability(permissions);
}
```

## 优势

1. **用户信任**：透明的权限使用增加用户信任
2. **安全性**：最小权限原则减少安全风险
3. **合规性**：符合浏览器扩展商店的政策要求
4. **用户体验**：在需要时请求权限提供更好的用户体验

## 实践建议

1. 只请求完成功能所必需的权限
2. 向用户清楚解释每个权限的用途
3. 在功能需要时才请求权限
4. 优雅处理权限被拒绝或撤销的情况
5. 定期审查权限使用情况