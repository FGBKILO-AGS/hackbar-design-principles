# 自定义载荷 (Custom Payloads)

## 概述

HackBar 允许用户创建和管理自定义载荷，以满足特定的渗透测试需求。这一功能提供了高度的可扩展性，使用户能够保存和快速使用常用的测试载荷。

## 核心思想

1. **用户定制**：允许用户创建和保存自己的载荷
2. **分类管理**：支持按类别组织载荷
3. **快速访问**：提供便捷的载荷使用接口
4. **持久化存储**：将载荷保存在浏览器存储中

## 代码示例

### 载荷数据结构

```typescript
// types/payload.ts
export interface CustomPayload {
  id: string;
  name: string;
  content: string;
  category: string;
  createdAt: number;
  updatedAt: number;
}

export interface PayloadCategory {
  id: string;
  name: string;
  description: string;
}
```

### 载荷存储管理

```typescript
// stores/custom-payload.ts
import { defineStore } from 'pinia';
import type { CustomPayload, PayloadCategory } from '@/types';

export const useCustomPayloadStore = defineStore('customPayload', () => {
  const payloads = ref<CustomPayload[]>([]);
  const categories = ref<PayloadCategory[]>([
    { id: 'general', name: '通用', description: '通用载荷' },
    { id: 'xss', name: 'XSS', description: '跨站脚本攻击载荷' },
    { id: 'sqli', name: 'SQLi', description: 'SQL注入载荷' },
    { id: 'lfi', name: 'LFI', description: '本地文件包含载荷' },
    { id: 'ssrf', name: 'SSRF', description: '服务器端请求伪造载荷' },
    { id: 'ssti', name: 'SSTI', description: '服务端模板注入载荷' }
  ]);
  
  // 从存储加载载荷
  const loadPayloads = async () => {
    try {
      const result = await chrome.storage.local.get(['customPayloads']);
      payloads.value = result.customPayloads || [];
    } catch (error) {
      console.error('Failed to load payloads:', error);
      payloads.value = [];
    }
  };
  
  // 保存载荷到存储
  const savePayloads = async () => {
    try {
      await chrome.storage.local.set({ 
        customPayloads: payloads.value 
      });
    } catch (error) {
      console.error('Failed to save payloads:', error);
    }
  };
  
  // 添加新载荷
  const addPayload = async (payload: Omit<CustomPayload, 'id' | 'createdAt' | 'updatedAt'>) => {
    const newPayload: CustomPayload = {
      id: generateId(),
      ...payload,
      createdAt: Date.now(),
      updatedAt: Date.now()
    };
    
    payloads.value.push(newPayload);
    await savePayloads();
    
    return newPayload;
  };
  
  // 更新载荷
  const updatePayload = async (id: string, updates: Partial<CustomPayload>) => {
    const index = payloads.value.findIndex(p => p.id === id);
    if (index !== -1) {
      payloads.value[index] = {
        ...payloads.value[index],
        ...updates,
        updatedAt: Date.now()
      };
      await savePayloads();
    }
  };
  
  // 删除载荷
  const deletePayload = async (id: string) => {
    payloads.value = payloads.value.filter(p => p.id !== id);
    await savePayloads();
  };
  
  // 按类别获取载荷
  const getPayloadsByCategory = (category: string) => {
    return payloads.value.filter(p => p.category === category);
  };
  
  // 搜索载荷
  const searchPayloads = (query: string) => {
    const lowerQuery = query.toLowerCase();
    return payloads.value.filter(p => 
      p.name.toLowerCase().includes(lowerQuery) || 
      p.content.toLowerCase().includes(lowerQuery)
    );
  };
  
  // 导出载荷
  const exportPayloads = (): string => {
    return JSON.stringify(payloads.value, null, 2);
  };
  
  // 导入载荷
  const importPayloads = async (data: string) => {
    try {
      const importedPayloads: CustomPayload[] = JSON.parse(data);
      
      // 验证导入的数据
      const validPayloads = importedPayloads.filter(p => 
        p.name && p.content && p.category
      );
      
      payloads.value = validPayloads;
      await savePayloads();
      
      return validPayloads.length;
    } catch (error) {
      console.error('Failed to import payloads:', error);
      throw new Error('Invalid payload data format');
    }
  };
  
  const generateId = (): string => {
    return Date.now().toString(36) + Math.random().toString(36).substr(2);
  };
  
  return {
    payloads: computed(() => payloads.value),
    categories: computed(() => categories.value),
    loadPayloads,
    addPayload,
    updatePayload,
    deletePayload,
    getPayloadsByCategory,
    searchPayloads,
    exportPayloads,
    importPayloads
  };
});
```

### 载荷管理界面

```vue
<!-- components/DialogCustomPayloadManagement.vue -->
<template>
  <v-dialog v-model="dialog" max-width="800px" fullscreen>
    <v-card>
      <v-card-title>
        <span class="text-h5">自定义载荷管理</span>
      </v-card-title>
      
      <v-card-text>
        <v-row>
          <v-col cols="12">
            <v-text-field
              v-model="searchQuery"
              label="搜索载荷"
              prepend-inner-icon="mdi-magnify"
              clearable
            />
          </v-col>
        </v-row>
        
        <v-row>
          <v-col cols="3">
            <v-list>
              <v-list-item
                v-for="category in categories"
                :key="category.id"
                :active="selectedCategory === category.id"
                @click="selectCategory(category.id)"
              >
                <v-list-item-content>
                  <v-list-item-title>{{ category.name }}</v-list-item-title>
                </v-list-item-content>
              </v-list-item>
            </v-list>
            
            <v-divider class="my-2" />
            
            <v-btn
              block
              color="primary"
              @click="showAddDialog = true"
            >
              添加载荷
            </v-btn>
            
            <v-btn
              block
              class="mt-2"
              @click="exportPayloads"
            >
              导出载荷
            </v-btn>
            
            <v-btn
              block
              class="mt-2"
              @click="triggerImport"
            >
              导入载荷
            </v-btn>
            
            <input
              ref="fileInput"
              type="file"
              accept=".json"
              style="display: none"
              @change="handleFileImport"
            >
          </v-col>
          
          <v-col cols="9">
            <v-data-table
              :headers="headers"
              :items="filteredPayloads"
              :items-per-page="10"
              class="elevation-1"
            >
              <template #item.actions="{ item }">
                <v-icon
                  small
                  class="mr-2"
                  @click="editPayload(item)"
                >
                  mdi-pencil
                </v-icon>
                <v-icon
                  small
                  @click="deletePayload(item.id)"
                >
                  mdi-delete
                </v-icon>
              </template>
              
              <template #item.content="{ item }">
                <div class="payload-preview">
                  {{ truncateContent(item.content) }}
                </div>
              </template>
            </v-data-table>
          </v-col>
        </v-row>
      </v-card-text>
      
      <v-card-actions>
        <v-spacer />
        <v-btn @click="closeDialog">关闭</v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
  
  <DialogCustomPayloadEdit
    v-model="showAddDialog"
    :payload="editingPayload"
    @save="handleSavePayload"
  />
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';
import { useCustomPayloadStore } from '@/stores/custom-payload';
import DialogCustomPayloadEdit from './DialogCustomPayloadEdit.vue';
import type { CustomPayload } from '@/types';

const props = defineProps<{
  modelValue: boolean;
}>();

const emit = defineEmits<{
  (e: 'update:modelValue', value: boolean): void;
}>();

// 响应式数据
const dialog = ref(false);
const searchQuery = ref('');
const selectedCategory = ref('all');
const showAddDialog = ref(false);
const editingPayload = ref<CustomPayload | null>(null);
const fileInput = ref<HTMLInputElement | null>(null);

// Store
const payloadStore = useCustomPayloadStore();

// 计算属性
const categories = computed(() => [
  { id: 'all', name: '全部' },
  ...payloadStore.categories
]);

const filteredPayloads = computed(() => {
  let result = payloadStore.payloads;
  
  // 按类别过滤
  if (selectedCategory.value !== 'all') {
    result = result.filter(p => p.category === selectedCategory.value);
  }
  
  // 按搜索查询过滤
  if (searchQuery.value) {
    const query = searchQuery.value.toLowerCase();
    result = result.filter(p => 
      p.name.toLowerCase().includes(query) || 
      p.content.toLowerCase().includes(query)
    );
  }
  
  return result;
});

const headers = [
  { text: '名称', value: 'name' },
  { text: '内容', value: 'content' },
  { text: '类别', value: 'category' },
  { text: '操作', value: 'actions', sortable: false }
];

// 方法
const truncateContent = (content: string): string => {
  return content.length > 50 ? content.substring(0, 50) + '...' : content;
};

const selectCategory = (category: string) => {
  selectedCategory.value = category;
};

const editPayload = (payload: CustomPayload) => {
  editingPayload.value = payload;
  showAddDialog.value = true;
};

const deletePayload = async (id: string) => {
  if (confirm('确定要删除这个载荷吗？')) {
    await payloadStore.deletePayload(id);
  }
};

const handleSavePayload = async (payload: CustomPayload) => {
  if (payload.id) {
    await payloadStore.updatePayload(payload.id, payload);
  } else {
    await payloadStore.addPayload(payload);
  }
  showAddDialog.value = false;
};

const exportPayloads = () => {
  const data = payloadStore.exportPayloads();
  const blob = new Blob([data], { type: 'application/json' });
  const url = URL.createObjectURL(blob);
  
  const a = document.createElement('a');
  a.href = url;
  a.download = `hackbar-payloads-${new Date().toISOString().slice(0, 10)}.json`;
  a.click();
  
  URL.revokeObjectURL(url);
};

const triggerImport = () => {
  fileInput.value?.click();
};

const handleFileImport = async (event: Event) => {
  const input = event.target as HTMLInputElement;
  const file = input.files?.[0];
  
  if (file) {
    try {
      const text = await file.text();
      const count = await payloadStore.importPayloads(text);
      alert(`成功导入 ${count} 个载荷`);
    } catch (error) {
      alert('导入失败：' + (error as Error).message);
    }
  }
  
  // 重置输入
  input.value = '';
};

const closeDialog = () => {
  dialog.value = false;
  emit('update:modelValue', false);
};

// 监听
watch(() => props.modelValue, (value) => {
  dialog.value = value;
  if (value) {
    payloadStore.loadPayloads();
  }
});

// 生命周期
onMounted(() => {
  dialog.value = props.modelValue;
});
</script>

<style scoped>
.payload-preview {
  font-family: 'Roboto Mono', monospace;
  font-size: 0.875rem;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
</style>
```

## 优势

1. **个性化**：用户可以创建符合自己需求的载荷
2. **组织性**：通过分类管理使载荷更易查找
3. **可移植**：支持导出和导入载荷
4. **易用性**：提供直观的管理界面

## 实践建议

1. 提供清晰的载荷分类系统
2. 支持载荷的搜索和过滤功能
3. 实现载荷的导入导出功能
4. 提供载荷预览功能
5. 确保载荷数据的安全性和完整性