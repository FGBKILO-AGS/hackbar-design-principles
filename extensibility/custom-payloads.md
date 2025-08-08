# 自定义载荷 (Custom Payloads)

## 原始定义

自定义载荷是指用户根据特定需求创建和管理的可重用数据片段。在渗透测试工具中，自定义载荷通常包含攻击向量、测试数据或特定场景下的输入数据，用户可以保存、组织和快速访问这些载荷以提高测试效率。

## 详细解释

HackBar 允许用户创建和管理自定义载荷，以满足特定的渗透测试需求。这一功能提供了高度的可扩展性，使用户能够保存和快速使用常用的测试载荷。自定义载荷系统不仅支持载荷的存储和检索，还提供了分类管理、搜索、导入导出等高级功能。

### 核心概念

1. **用户定制**：允许用户创建和保存自己的载荷
2. **分类管理**：支持按类别组织载荷
3. **快速访问**：提供便捷的载荷使用接口
4. **持久化存储**：将载荷保存在浏览器存储中
5. **数据安全**：确保载荷数据的安全性和完整性

### 在 HackBar 中的应用

HackBar 允许用户创建和管理自定义载荷，以满足特定的渗透测试需求。这一功能提供了高度的可扩展性，使用户能够保存和快速使用常用的测试载荷。

## 实施步骤

### 第一步：载荷数据结构定义

定义自定义载荷的数据结构和相关类型。

```typescript
// types/payload.ts
export interface CustomPayload {
  id: string;
  name: string;
  content: string;
  category: string;
  description?: string;
  tags?: string[];
  createdAt: number;
  updatedAt: number;
  usageCount: number;
}

export interface PayloadCategory {
  id: string;
  name: string;
  description: string;
  icon?: string;
}

export interface PayloadImportExportData {
  version: string;
  exportedAt: number;
  payloads: CustomPayload[];
  categories: PayloadCategory[];
}
```

### 第二步：载荷存储管理实现

实现载荷的存储、检索和管理功能。

```typescript
// stores/custom-payload.ts
import { defineStore } from 'pinia';
import type { CustomPayload, PayloadCategory, PayloadImportExportData } from '@/types';
import { performanceMonitor } from '@/utils/performance-monitor';

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
    return performanceMonitor.timeAsync('loadPayloads', async () => {
      try {
        const result = await chrome.storage.local.get(['customPayloads', 'payloadCategories']);
        payloads.value = result.customPayloads || [];
        if (result.payloadCategories) {
          categories.value = result.payloadCategories;
        }
      } catch (error) {
        console.error('Failed to load payloads:', error);
        payloads.value = [];
      }
    });
  };
  
  // 保存载荷到存储
  const savePayloads = async () => {
    return performanceMonitor.timeAsync('savePayloads', async () => {
      try {
        await chrome.storage.local.set({ 
          customPayloads: payloads.value,
          payloadCategories: categories.value
        });
      } catch (error) {
        console.error('Failed to save payloads:', error);
      }
    });
  };
  
  // 添加新载荷
  const addPayload = async (payload: Omit<CustomPayload, 'id' | 'createdAt' | 'updatedAt' | 'usageCount'>) => {
    return performanceMonitor.timeAsync('addPayload', async () => {
      const newPayload: CustomPayload = {
        id: generateId(),
        ...payload,
        createdAt: Date.now(),
        updatedAt: Date.now(),
        usageCount: 0
      };
      
      // 验证载荷数据
      if (!validatePayload(newPayload)) {
        throw new Error('Invalid payload data');
      }
      
      // 清理载荷数据
      const cleanedPayload = sanitizePayload(newPayload);
      
      payloads.value.push(cleanedPayload);
      await savePayloads();
      
      return cleanedPayload;
    });
  };
  
  // 更新载荷
  const updatePayload = async (id: string, updates: Partial<CustomPayload>) => {
    return performanceMonitor.timeAsync('updatePayload', async () => {
      const index = payloads.value.findIndex(p => p.id === id);
      if (index !== -1) {
        payloads.value[index] = {
          ...payloads.value[index],
          ...updates,
          updatedAt: Date.now()
        };
        
        // 验证更新后的载荷
        if (!validatePayload(payloads.value[index])) {
          throw new Error('Invalid payload data after update');
        }
        
        await savePayloads();
      }
    });
  };
  
  // 增加使用计数
  const incrementUsageCount = async (id: string) => {
    const payload = payloads.value.find(p => p.id === id);
    if (payload) {
      await updatePayload(id, { usageCount: payload.usageCount + 1 });
    }
  };
  
  // 删除载荷
  const deletePayload = async (id: string) => {
    return performanceMonitor.timeAsync('deletePayload', async () => {
      payloads.value = payloads.value.filter(p => p.id !== id);
      await savePayloads();
    });
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
      p.content.toLowerCase().includes(lowerQuery) ||
      (p.description && p.description.toLowerCase().includes(lowerQuery)) ||
      (p.tags && p.tags.some(tag => tag.toLowerCase().includes(lowerQuery)))
    );
  };
  
  // 导出载荷
  const exportPayloads = (): string => {
    const exportData: PayloadImportExportData = {
      version: '1.0',
      exportedAt: Date.now(),
      payloads: payloads.value,
      categories: categories.value
    };
    
    return JSON.stringify(exportData, null, 2);
  };
  
  // 导入载荷
  const importPayloads = async (data: string) => {
    return performanceMonitor.timeAsync('importPayloads', async () => {
      try {
        const importedData: PayloadImportExportData = JSON.parse(data);
        
        // 验证导入的数据
        if (!importedData.version || !importedData.payloads) {
          throw new Error('Invalid payload data format');
        }
        
        // 验证和清理载荷
        const validPayloads = importedData.payloads
          .filter(p => validatePayload(p))
          .map(p => sanitizePayload(p));
        
        payloads.value = validPayloads;
        
        // 如果有分类数据，也导入分类
        if (importedData.categories) {
          categories.value = importedData.categories;
        }
        
        await savePayloads();
        
        return validPayloads.length;
      } catch (error) {
        console.error('Failed to import payloads:', error);
        throw new Error('Invalid payload data format');
      }
    });
  };
  
  // 添加分类
  const addCategory = async (category: Omit<PayloadCategory, 'id'>) => {
    const newCategory: PayloadCategory = {
      id: generateId(),
      ...category
    };
    
    categories.value.push(newCategory);
    await savePayloads();
    
    return newCategory;
  };
  
  // 验证载荷数据
  const validatePayload = (payload: CustomPayload): boolean => {
    // 检查必需字段
    if (!payload.name || !payload.content || !payload.category) {
      return false;
    }
    
    // 检查字段长度
    if (payload.name.length > 100 || payload.content.length > 10000) {
      return false;
    }
    
    // 检查时间戳
    if (payload.createdAt > Date.now() || payload.updatedAt > Date.now()) {
      return false;
    }
    
    // 检查使用计数
    if (payload.usageCount < 0) {
      return false;
    }
    
    return true;
  };
  
  // 清理载荷数据
  const sanitizePayload = (payload: CustomPayload): CustomPayload => {
    return {
      ...payload,
      name: sanitizeString(payload.name),
      content: sanitizeString(payload.content),
      description: payload.description ? sanitizeString(payload.description) : undefined,
      tags: payload.tags ? payload.tags.map(tag => sanitizeString(tag)) : undefined
    };
  };
  
  // 字符串清理
  const sanitizeString = (str: string): string => {
    // 移除控制字符
    return str.replace(/[\x00-\x08\x0B\x0C\x0E-\x1F\x7F]/g, '');
  };
  
  const generateId = (): string => {
    return Date.now().toString(36) + Math.random().toString(36).substr(2);
  };
  
  // 获取使用统计
  const getUsageStats = () => {
    const totalPayloads = payloads.value.length;
    const totalUsage = payloads.value.reduce((sum, p) => sum + p.usageCount, 0);
    const mostUsed = [...payloads.value].sort((a, b) => b.usageCount - a.usageCount).slice(0, 5);
    
    return {
      totalPayloads,
      totalUsage,
      mostUsed
    };
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
    importPayloads,
    addCategory,
    incrementUsageCount,
    getUsageStats
  };
});
```

### 第三步：载荷管理界面实现

创建用户友好的载荷管理界面。

```vue
<!-- components/DialogCustomPayloadManagement.vue -->
<template>
  <v-dialog v-model="dialog" max-width="1200px" fullscreen>
    <v-card>
      <v-card-title>
        <span class="text-h5">自定义载荷管理</span>
        <v-spacer />
        <v-btn icon @click="closeDialog">
          <v-icon>mdi-close</v-icon>
        </v-btn>
      </v-card-title>
      
      <v-card-text>
        <v-row>
          <v-col cols="12">
            <v-text-field
              v-model="searchQuery"
              label="搜索载荷"
              prepend-inner-icon="mdi-magnify"
              clearable
              @keyup.enter="performSearch"
            />
          </v-col>
        </v-row>
        
        <v-row>
          <v-col cols="3">
            <v-list dense>
              <v-list-item
                :active="selectedCategory === 'all'"
                @click="selectCategory('all')"
              >
                <v-list-item-content>
                  <v-list-item-title>全部载荷</v-list-item-title>
                  <v-list-item-subtitle>{{ totalPayloads }} 个项目</v-list-item-subtitle>
                </v-list-item-content>
              </v-list-item>
              
              <v-divider />
              
              <v-list-item
                v-for="category in categories"
                :key="category.id"
                :active="selectedCategory === category.id"
                @click="selectCategory(category.id)"
              >
                <v-list-item-icon>
                  <v-icon>{{ category.icon || 'mdi-folder' }}</v-icon>
                </v-list-item-icon>
                <v-list-item-content>
                  <v-list-item-title>{{ category.name }}</v-list-item-title>
                  <v-list-item-subtitle>
                    {{ getPayloadsByCategory(category.id).length }} 个项目
                  </v-list-item-subtitle>
                </v-list-item-content>
              </v-list-item>
            </v-list>
            
            <v-divider class="my-2" />
            
            <v-btn
              block
              color="primary"
              @click="showAddDialog = true"
            >
              <v-icon left>mdi-plus</v-icon>
              添加载荷
            </v-btn>
            
            <v-btn
              block
              class="mt-2"
              @click="exportPayloads"
            >
              <v-icon left>mdi-export</v-icon>
              导出载荷
            </v-btn>
            
            <v-btn
              block
              class="mt-2"
              @click="triggerImport"
            >
              <v-icon left>mdi-import</v-icon>
              导入载荷
            </v-btn>
            
            <v-btn
              block
              class="mt-2"
              @click="showStats = true"
            >
              <v-icon left>mdi-chart-bar</v-icon>
              使用统计
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
              :items-per-page="15"
              :loading="loading"
              class="elevation-1"
              @click:row="handleRowClick"
            >
              <template #item.name="{ item }">
                <div class="payload-name">
                  <strong>{{ item.name }}</strong>
                  <div class="payload-description" v-if="item.description">
                    {{ item.description }}
                  </div>
                </div>
              </template>
              
              <template #item.category="{ item }">
                <v-chip small :color="getCategoryColor(item.category)">
                  {{ getCategoryName(item.category) }}
                </v-chip>
              </template>
              
              <template #item.usageCount="{ item }">
                <v-chip small outlined>
                  {{ item.usageCount }}
                </v-chip>
              </template>
              
              <template #item.actions="{ item }">
                <v-icon
                  small
                  class="mr-2"
                  @click.stop="editPayload(item)"
                >
                  mdi-pencil
                </v-icon>
                <v-icon
                  small
                  @click.stop="deletePayload(item.id)"
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
    </v-card>
  </v-dialog>
  
  <DialogCustomPayloadEdit
    v-model="showAddDialog"
    :payload="editingPayload"
    @save="handleSavePayload"
  />
  
  <v-dialog v-model="showStats" max-width="600px">
    <v-card>
      <v-card-title>使用统计</v-card-title>
      <v-card-text>
        <v-list>
          <v-list-item>
            <v-list-item-content>
              <v-list-item-title>总载荷数</v-list-item-title>
              <v-list-item-subtitle>{{ usageStats.totalPayloads }}</v-list-item-subtitle>
            </v-list-item-content>
          </v-list-item>
          <v-list-item>
            <v-list-item-content>
              <v-list-item-title>总使用次数</v-list-item-title>
              <v-list-item-subtitle>{{ usageStats.totalUsage }}</v-list-item-subtitle>
            </v-list-item-content>
          </v-list-item>
          <v-list-item>
            <v-list-item-content>
              <v-list-item-title>平均使用次数</v-list-item-title>
              <v-list-item-subtitle>
                {{ usageStats.totalPayloads > 0 ? (usageStats.totalUsage / usageStats.totalPayloads).toFixed(2) : 0 }}
              </v-list-item-subtitle>
            </v-list-item-content>
          </v-list-item>
        </v-list>
        
        <v-subheader>最常用载荷</v-subheader>
        <v-list>
          <v-list-item
            v-for="(payload, index) in usageStats.mostUsed"
            :key="payload.id"
          >
            <v-list-item-avatar>
              {{ index + 1 }}
            </v-list-item-avatar>
            <v-list-item-content>
              <v-list-item-title>{{ payload.name }}</v-list-item-title>
              <v-list-item-subtitle>
                使用 {{ payload.usageCount }} 次
              </v-list-item-subtitle>
            </v-list-item-content>
          </v-list-item>
        </v-list>
      </v-card-text>
      <v-card-actions>
        <v-spacer />
        <v-btn @click="showStats = false">关闭</v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, watch } from 'vue';
import { useCustomPayloadStore } from '@/stores/custom-payload';
import DialogCustomPayloadEdit from './DialogCustomPayloadEdit.vue';
import type { CustomPayload } from '@/types';

const props = defineProps<{
  modelValue: boolean;
}>();

const emit = defineEmits<{
  (e: 'update:modelValue', value: boolean): void;
  (e: 'use-payload', payload: CustomPayload): void;
}>();

// 响应式数据
const dialog = ref(false);
const searchQuery = ref('');
const selectedCategory = ref('all');
const showAddDialog = ref(false);
const editingPayload = ref<CustomPayload | null>(null);
const fileInput = ref<HTMLInputElement | null>(null);
const showStats = ref(false);
const loading = ref(false);

// Store
const payloadStore = useCustomPayloadStore();

// 计算属性
const totalPayloads = computed(() => payloadStore.payloads.length);

const filteredPayloads = computed(() => {
  let result = payloadStore.payloads;
  
  // 按类别过滤
  if (selectedCategory.value !== 'all') {
    result = result.filter(p => p.category === selectedCategory.value);
  }
  
  // 按搜索查询过滤
  if (searchQuery.value) {
    result = payloadStore.searchPayloads(searchQuery.value);
  }
  
  // 按使用次数排序
  return [...result].sort((a, b) => b.usageCount - a.usageCount);
});

const headers = [
  { text: '名称', value: 'name', width: '25%' },
  { text: '内容', value: 'content', width: '40%' },
  { text: '类别', value: 'category', width: '15%' },
  { text: '使用次数', value: 'usageCount', width: '10%' },
  { text: '操作', value: 'actions', sortable: false, width: '10%' }
];

const usageStats = computed(() => payloadStore.getUsageStats());

// 方法
const truncateContent = (content: string): string => {
  return content.length > 100 ? content.substring(0, 100) + '...' : content;
};

const getCategoryName = (categoryId: string): string => {
  const category = payloadStore.categories.find(c => c.id === categoryId);
  return category ? category.name : categoryId;
};

const getCategoryColor = (categoryId: string): string => {
  const colors: Record<string, string> = {
    'general': 'blue',
    'xss': 'red',
    'sqli': 'orange',
    'lfi': 'purple',
    'ssrf': 'green',
    'ssti': 'pink'
  };
  
  return colors[categoryId] || 'grey';
};

const selectCategory = (category: string) => {
  selectedCategory.value = category;
};

const editPayload = (payload: CustomPayload) => {
  editingPayload.value = payload;
  showAddDialog.value = true;
};

const deletePayload = async (id: string) => {
  const confirmed = confirm('确定要删除这个载荷吗？');
  if (confirmed) {
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
      loading.value = true;
      const text = await file.text();
      const count = await payloadStore.importPayloads(text);
      alert(`成功导入 ${count} 个载荷`);
    } catch (error) {
      alert('导入失败：' + (error as Error).message);
    } finally {
      loading.value = false;
    }
  }
  
  // 重置输入
  input.value = '';
};

const performSearch = () => {
  // 搜索已经在 computed 中自动执行
};

const handleRowClick = (item: CustomPayload) => {
  // 增加使用计数
  payloadStore.incrementUsageCount(item.id);
  
  // 发出使用载荷事件
  emit('use-payload', item);
};

const closeDialog = () => {
  dialog.value = false;
  emit('update:modelValue', false);
};

// 监听
watch(() => props.modelValue, (value) => {
  dialog.value = value;
  if (value) {
    loading.value = true;
    payloadStore.loadPayloads().finally(() => {
      loading.value = false;
    });
  }
});

// 生命周期
onMounted(() => {
  dialog.value = props.modelValue;
});
</script>

<style scoped>
.payload-name strong {
  display: block;
}

.payload-description {
  font-size: 0.875rem;
  color: rgba(0, 0, 0, 0.6);
  margin-top: 4px;
}

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
5. **可追踪性**：记录载荷使用情况
6. **安全性**：对载荷内容进行验证和清理

## 最佳实践

1. **提供清晰的载荷分类系统**：建立合理的分类体系便于管理
2. **支持载荷的搜索和过滤功能**：实现高效的载荷检索
3. **实现载荷的导入导出功能**：支持载荷的备份和共享
4. **提供载荷预览功能**：让用户快速了解载荷内容
5. **确保载荷数据的安全性和完整性**：实施数据验证和清理机制
6. **记录载荷使用统计**：帮助用户了解载荷使用情况
7. **支持载荷的版本管理**：允许用户跟踪载荷变更历史
8. **提供批量操作功能**：支持批量导入、导出和删除载荷

## 总结

自定义载荷系统是 HackBar 扩展性的重要体现，它允许用户根据自己的需求创建、管理和使用测试载荷。通过完善的存储管理、用户友好的界面设计和丰富的功能特性，用户可以高效地组织和重用常用的测试载荷。这一系统不仅提高了渗透测试的效率，还增强了工具的灵活性和实用性。