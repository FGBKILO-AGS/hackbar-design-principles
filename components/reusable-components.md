# 可复用组件 (Reusable Components)

## 概述

在 HackBar 中，我们设计了许多可复用的组件，以提高开发效率并保持界面一致性。这些组件遵循设计规范，可以在不同功能模块中重复使用。

## 核心思想

1. **通用性**：组件设计考虑多种使用场景
2. **配置化**：通过 props 提供灵活的配置选项
3. **一致性**：遵循统一的设计风格和交互模式
4. **低耦合**：组件不依赖具体业务逻辑

## 代码示例

### 菜单组件

```vue
<!-- components/MenuEncoding.vue -->
<template>
  <v-menu>
    <template #activator="{ props }">
      <v-btn v-bind="props" variant="text">
        <v-icon>{{ mdiContentCopy }}</v-icon>
        Encoding
      </v-btn>
    </template>
    <v-list>
      <v-list-item 
        v-for="item in encodingOptions" 
        :key="item.value"
        @click="handleEncoding(item.value)"
      >
        <v-list-item-title>{{ item.title }}</v-list-item-title>
      </v-list-item>
    </v-list>
  </v-menu>
</template>

<script setup lang="ts">
import { mdiContentCopy } from '@mdi/js';
import { ref } from 'vue';

// 可配置的选项
const encodingOptions = ref([
  { title: 'URL Encode', value: 'url-encode' },
  { title: 'URL Decode', value: 'url-decode' },
  { title: 'Base64 Encode', value: 'base64-encode' },
  { title: 'Base64 Decode', value: 'base64-decode' },
  { title: 'Hex Encode', value: 'hex-encode' },
  { title: 'Hex Decode', value: 'hex-decode' }
]);

// 通过 emit 与父组件通信
const emit = defineEmits<{
  (e: 'encode', type: string): void
}>();

const handleEncoding = (type: string) => {
  emit('encode', type);
};
</script>
```

### 对话框组件

```vue
<!-- components/DialogCustomPayloadEdit.vue -->
<template>
  <v-dialog v-model="dialog" max-width="600px">
    <v-card>
      <v-card-title>
        <span class="text-h5">{{ isEditing ? 'Edit' : 'Add' }} Custom Payload</span>
      </v-card-title>
      <v-card-text>
        <v-form ref="form" v-model="valid">
          <v-text-field
            v-model="payloadData.name"
            label="Name*"
            :rules="nameRules"
            required
          />
          <v-textarea
            v-model="payloadData.content"
            label="Content*"
            :rules="contentRules"
            required
          />
          <v-select
            v-model="payloadData.category"
            :items="categories"
            label="Category"
          />
        </v-form>
      </v-card-text>
      <v-card-actions>
        <v-spacer />
        <v-btn @click="closeDialog">Cancel</v-btn>
        <v-btn 
          :disabled="!valid" 
          color="primary" 
          @click="savePayload"
        >
          Save
        </v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<script setup lang="ts">
import { ref, reactive, watch } from 'vue';

// Props 定义
const props = defineProps<{
  modelValue: boolean;
  payload?: CustomPayload;
}>();

// Emit 定义
const emit = defineEmits<{
  (e: 'update:modelValue', value: boolean): void;
  (e: 'save', payload: CustomPayload): void;
}>();

// 响应式数据
const dialog = ref(false);
const valid = ref(false);
const isEditing = ref(false);

const payloadData = reactive<CustomPayload>({
  id: '',
  name: '',
  content: '',
  category: 'general'
});

const categories = ['general', 'xss', 'sqli', 'lfi', 'ssrf', 'ssti'];

// 验证规则
const nameRules = [(v: string) => !!v || 'Name is required'];
const contentRules = [(v: string) => !!v || 'Content is required'];

// 监听 modelValue 变化
watch(() => props.modelValue, (newValue) => {
  dialog.value = newValue;
  if (newValue && props.payload) {
    isEditing.value = true;
    Object.assign(payloadData, props.payload);
  } else {
    isEditing.value = false;
    resetForm();
  }
});

// 方法定义
const closeDialog = () => {
  dialog.value = false;
  emit('update:modelValue', false);
};

const resetForm = () => {
  payloadData.id = '';
  payloadData.name = '';
  payloadData.content = '';
  payloadData.category = 'general';
};

const savePayload = () => {
  if (valid.value) {
    emit('save', { ...payloadData });
    closeDialog();
  }
};
</script>
```

### 工具函数组件

```vue
<!-- components/PrettyRawResponse.vue -->
<template>
  <div class="response-viewer">
    <v-toolbar dense>
      <v-spacer />
      <v-btn 
        icon 
        @click="toggleFormat"
      >
        <v-icon>{{ formatted ? mdiCodeBraces : mdiCodeTags }}</v-icon>
      </v-btn>
      <v-btn 
        icon 
        @click="copyResponse"
      >
        <v-icon>{{ mdiContentCopy }}</v-icon>
      </v-btn>
    </v-toolbar>
    <v-card-text class="response-content">
      <pre v-if="formatted"><code v-html="highlightedResponse" /></pre>
      <pre v-else><code>{{ rawResponse }}</code></pre>
    </v-card-text>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue';
import { useClipboard } from '@vueuse/core';
import hljs from 'highlight.js';
import {
  mdiCodeBraces,
  mdiCodeTags,
  mdiContentCopy
} from '@mdi/js';

// Props 定义
const props = defineProps<{
  response: string;
  language?: string;
}>();

// 响应式数据
const formatted = ref(true);
const rawResponse = computed(() => props.response);

// 计算属性
const highlightedResponse = computed(() => {
  if (props.language) {
    return hljs.highlight(props.response, { language: props.language }).value;
  }
  return hljs.highlightAuto(props.response).value;
});

// 方法定义
const toggleFormat = () => {
  formatted.value = !formatted.value;
};

const copyResponse = () => {
  const { copy } = useClipboard();
  copy(props.response);
};
</script>

<style scoped>
.response-viewer {
  height: 100%;
  display: flex;
  flex-direction: column;
}

.response-content {
  flex: 1;
  overflow: auto;
  padding: 16px;
}

pre {
  margin: 0;
  white-space: pre-wrap;
  word-wrap: break-word;
}
</style>
```

## 优势

1. **提高开发效率**：复用已有组件，减少重复开发
2. **保证一致性**：统一的组件外观和交互行为
3. **易于维护**：集中管理组件逻辑，一处修改多处生效
4. **降低耦合**：组件与业务逻辑分离

## 实践建议

1. 识别可复用的 UI 元素并抽象为组件
2. 提供丰富的配置选项以适应不同场景
3. 编写清晰的文档说明组件用法
4. 建立组件库方便团队共享
5. 定期审查和优化组件设计