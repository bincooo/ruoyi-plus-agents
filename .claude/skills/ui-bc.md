---
name: ui-bc
description: 当开发 bell-plus 前端 UI 时使用 —— antdv-next 组件、vxe-table 表格、useVbenModal 弹窗、字典标签。
---

# UI Components - antdv-next + vxe-table 组件封装规范

## 职责范围

规范 RuoYi-Vue-Plus（bell-plus 前端）中 **antdv-next**（Ant Design Vue Next）、**vxe-table** 表格、以及基于 **vben-admin** 的 `@/components` 封装组件的使用与二次封装，涵盖表单、表格、弹窗、字典标签等常见场景。

> ⚠️ 本项目前端为 Vue 3 + TypeScript + Vite + Pinia + **antdv-next**（不是 Element Plus）。表格基于 **vxe-table**，弹窗基于 vben 的 `useVbenModal`。组件通过 `@antdv-next/auto-import-resolver` 自动按需导入，`a-*` 标签可直接使用；需要类型或具名组件时从 `antdv-next` 显式 import。

---

## 核心规范

### 1. 组件来源与导入

| 用途 | 来源 | 示例 |
|------|------|------|
| 基础 UI（Button/Popconfirm/Space/Spin/Switch 等） | `antdv-next` | `import { Popconfirm, Space, Spin } from 'antdv-next'` |
| 表单基础 | `antdv-next` | `import { Form, FormItem } from 'antdv-next'`；类型 `import type { FormInstance } from 'antdv-next'` |
| 表单封装组件 | `@/components/global/form` | `FormInput`, `FormSelect`, `FormInputNumber`, `FormTextArea`（具名导入） |
| 表格 | `@/components/vxe-table` + `vxe-table` | `VxeGrid`, `withDefaultVxeGridOptions`, `useTableQuery`, `resolveQueryFormValues` |
| 弹窗 Hook | `@/components` | `useVbenModal` |
| 字典 | `@/utils/dict` | `getDictOptions(DictEnum)` |
| i18n | `@/locales` | `$t('pages.common.edit')` |
| 表单校验类型 | `@/types/form` | `import type { AntdFormRules } from '@/types/form'` |

### 2. 组件封装原则

- **单一职责**：搜索表单、表格、编辑弹窗各为独立 `.vue`，互不耦合。
- **类型先行**：用业务实体类型（如 `Role`）泛型化 `withDefaultVxeGridOptions<Role>()`，避免 `any`。
- **vben 封装优先**：表单字段、弹窗、表格均走 `@/components` 已封装的能力，不要直接手搓原生 antdv 表单布局。
- **字典统一**：状态/类型字段统一用 `DictEnum` + `getDictOptions` + `OptionsTag` 展示，不要散落硬编码。

---

### 3. 表格组件（VxeGrid）

表格统一使用 `vxe-table` 的 `VxeGrid`，并通过 `@/components/vxe-table` 的封装工具配置。

```vue
<!-- role.vue - 列表页表格 -->
<script setup lang="ts">
import type { Role } from '@/api/system/role/model';
import type { VxeGridInstance, VxeGridListeners } from 'vxe-table';

import { ref, useTemplateRef } from 'vue';
import { roleList, roleRemove, roleChangeStatus } from '@/api/system/role';
import { useAccess } from '@/components/access';
import {
  resolveQueryFormValues,
  useTableQuery,
  withDefaultVxeGridOptions,
} from '@/components/vxe-table';
import { Popconfirm, Space } from 'antdv-next';
import { VxeGrid } from 'vxe-table';

import { columns } from './data';

const tableRef = useTemplateRef<VxeGridInstance<Role>>('tableRef');

const gridOptions = withDefaultVxeGridOptions<Role>({
  columns,
  checkboxConfig: { highlight: true }, // 翻页保留选中
  pagerConfig: { enabled: true },
  proxyConfig: { ajax: { query: ({ page }) => roleList({ ...page }) } },
});

const gridEvents: VxeGridListeners = {
  // 统一在 listeners 里分发事件，避免模板里堆 @click
};

// 查询：把搜索表单值合并进 grid 的查询请求
const { query, reload } = useTableQuery(tableRef);

function handleSearch(form: Record<string, any>) {
  query(resolveQueryFormValues(form));
}

async function handleRemove(row: Role) {
  await roleRemove(row.id);
  reload();
}
</script>

<template>
  <VxeGrid ref="tableRef" v-bind="gridOptions" v-on="gridEvents">
    <template #toolbar-actions>
      <RoleSearchForm @search="handleSearch" />
    </template>
    <template #action="{ row }">
      <Popconfirm :title="`确认删除角色「${row.roleName}」？`" @confirm="handleRemove(row)">
        <a-button danger type="link">删除</a-button>
      </Popconfirm>
    </template>
  </VxeGrid>
</template>
```

要点：
- 列定义抽到 `./data.ts`（`columns`），不要堆在模板里。
- 请求统一走 `proxyConfig.ajax.query`，**禁止**在组件里 `onMounted` 手动调接口再赋值 `data`。
- 表格实例用 `useTemplateRef<VxeGridInstance<Role>>('tableRef')`，不要 `ref<VxeGridInstance>()` 后再 `.value`。

---

### 4. 表单组件（antdv-next + vben 封装）

#### 4.1 编辑弹窗表单

表单字段统一用 `@/components/global/form` 的封装组件（`FormInput` / `FormSelect` / `FormInputNumber` / `FormTextArea`），布局用 `Form` + `FormItem`。

```vue
<!-- role-modal.vue -->
<script setup lang="ts">
import type { Role } from '@/api/system/role/model';
import type { AntdFormRules } from '@/types/form';
import type { FormInstance } from 'antdv-next';

import { computed, ref } from 'vue';
import { roleAdd, roleInfo, roleUpdate } from '@/api/system/role';
import { useVbenModal } from '@/components';
import {
  FormInput as Input,
  FormInputNumber as InputNumber,
  FormSelect as Select,
  FormTextArea as TextArea,
} from '@/components/global/form';
import { DictEnum } from '@/constants';
import { $t } from '@/locales';
import { getDictOptions } from '@/utils/dict';
import { Form, FormItem } from 'antdv-next';

const emit = defineEmits<{ reload: [] }>();
const isUpdate = ref(false);
const title = computed(() => (isUpdate.value ? $t('pages.common.edit') : $t('pages.common.add')));

type FormData = Partial<Role>;

function getDefaultValues(): FormData {
  return { roleId: undefined, roleName: '', roleKey: '', roleSort: 1, status: '0', remark: '' };
}

const formData = ref<FormData>(getDefaultValues());
const formInstance = ref<FormInstance>();

const formRules = ref<AntdFormRules<FormData>>({
  roleName: [{ required: true, message: $t('ui.formRules.required') }],
  roleKey: [{ required: true, message: $t('ui.formRules.required') }],
  roleSort: [{ required: true, message: $t('ui.formRules.required') }],
  status: [{ required: true, message: $t('ui.formRules.selectRequired') }],
});

const [Modal, modalApi] = useVbenModal({
  onOpenChange: async (open) => {
    if (!open) return;
    const data = modalApi.getData<Role>();
    if (data?.roleId) {
      isUpdate.value = true;
      Object.assign(formData.value, await roleInfo(data.roleId));
    } else {
      isUpdate.value = false;
      formData.value = getDefaultValues();
    }
  },
  onConfirm: async () => {
    await formInstance.value?.validate();
    isUpdate.value ? await roleUpdate(formData.value) : await roleAdd(formData.value);
    modalApi.close();
    emit('reload');
  },
});
</script>

<template>
  <Modal :title="title">
    <Form ref="formInstance" :model="formData" :rules="formRules" layout="vertical">
      <FormItem name="roleName" label="角色名称">
        <Input v-model:value="formData.roleName" placeholder="请输入角色名称" />
      </FormItem>
      <FormItem name="roleKey" label="权限字符">
        <Input v-model:value="formData.roleKey" placeholder="请输入权限字符" />
      </FormItem>
      <FormItem name="status" label="状态">
        <Select v-model:value="formData.status" :options="getDictOptions(DictEnum.SYS_NORMAL_DISABLE)" />
      </FormItem>
      <FormItem name="roleSort" label="显示顺序">
        <InputNumber v-model:value="formData.roleSort" :min="0" />
      </FormItem>
      <FormItem name="remark" label="备注">
        <TextArea v-model:value="formData.remark" :rows="2" />
      </FormItem>
    </Form>
  </Modal>
</template>
```

#### 4.2 搜索表单

搜索表单独立成组件，emit `search` 事件给列表页，**不要**直接持有表格实例。

```vue
<!-- role-search.vue -->
<script setup lang="ts">
import { reactive } from 'vue';
import { FormInput as Input, FormSelect as Select } from '@/components/global/form';
import { DictEnum } from '@/constants';
import { getDictOptions } from '@/utils/dict';
import { Form, FormItem } from 'antdv-next';
import { SearchButtonGroup } from '@/components/table';

const emit = defineEmits<{ search: [form: Record<string, any>]; reset: [] }>();
const form = reactive({ roleName: '', status: undefined });

function handleSearch() { emit('search', { ...form }); }
function handleReset() { Object.assign(form, { roleName: '', status: undefined }); emit('reset'); }
</script>

<template>
  <Form :model="form" class="table-search-grid">
    <FormItem name="roleName" label="角色名称">
      <Input v-model:value="form.roleName" allow-clear @press-enter="handleSearch" />
    </FormItem>
    <FormItem name="status" label="状态">
      <Select v-model:value="form.status" :options="getDictOptions(DictEnum.SYS_NORMAL_DISABLE)" allow-clear />
    </FormItem>
    <SearchButtonGroup @search="handleSearch" @reset="handleReset" />
  </Form>
</template>
```

---

### 5. 弹窗（useVbenModal）

**禁止**用 antdv 原生 `Modal` 直接 `v-model:open` 管开关；统一用 vben 的 `useVbenModal` hook：

```ts
const [Modal, modalApi] = useVbenModal({
  title: '编辑角色',
  onOpenChange: (open) => { /* 打开/关闭回调，可在此拉详情 */ },
  onConfirm: async () => { /* 提交校验+接口调用，成功后 modalApi.close() */ },
});

// 打开方式（列表页）：
modalApi.setData({ roleId: row.roleId });
modalApi.open();
```

- 列表页持有 `modalApi`，编辑组件持有 `[Modal]` 与 `onConfirm`。
- 关闭后通过 `emit('reload')` 触发表格 `reload()`，**不要**在弹窗里直接操作表格实例。

---

### 6. 字典标签展示

状态字段在表格列里统一用 `OptionsTag`（`@/components/table`）渲染，不要逐行写 `<a-tag>` 判颜色：

```ts
// data.ts
import { DictEnum } from '@/constants';
import { OptionsTag } from '@/components/table';
import { h } from 'vue';

export const columns: VxeColumnSlotTypes[] = [
  { field: 'roleName', title: '角色名称' },
  {
    field: 'status',
    title: '状态',
    slots: {
      default: ({ row }) => h(OptionsTag, { options: getDictOptions(DictEnum.SYS_NORMAL_DISABLE), value: row.status }),
    },
  },
];
```

---

## 组件命名规范

| 组件名 | 说明 | 来源 |
|--------|------|------|
| `VxeGrid` | 列表表格 | vxe-table（封装于 `@/components/vxe-table`） |
| `Form / FormItem` | 表单容器与项 | antdv-next |
| `FormInput / FormSelect / FormInputNumber / FormTextArea` | 表单字段封装 | `@/components/global/form` |
| `OptionsTag` | 字典状态标签 | `@/components/table` |
| `SearchButtonGroup` | 搜索/重置按钮组 | `@/components/table` |
| `useVbenModal` | 弹窗 Hook | `@/components` |
| `ApiSwitch` | 异步状态开关 | `@/components/global` |

---

## 常见错误

| 错误 | 正确做法 |
|------|----------|
| 用 `el-table` / `el-form` / `el-input` | 本项目无 Element Plus，改用 `VxeGrid` + antdv-next 封装组件 |
| 表格 `data` 手动赋值、`onMounted` 调接口 | 用 `VxeGrid` 的 `proxyConfig.ajax.query` + `useTableQuery` |
| 弹窗用 `v-model:open` 手动管开关 | 用 `useVbenModal`，列表页 `modalApi.open()` |
| 校验规则写 `as FormRules` | 用 `AntdFormRules<T>` 泛型，字段可被类型检查 |
| 硬编码状态文案/颜色 | 走 `DictEnum` + `OptionsTag` |
| 组件实例 `ref<VxeGridInstance>()` | 用 `useTemplateRef<VxeGridInstance<Role>>('tableRef')` |

---

## 触发关键词

- antdv-next / Ant Design Vue
- vxe-table / VxeGrid
- vben / useVbenModal
- 表单组件 / 表格组件 / 弹窗
- 字典标签 / OptionsTag
- 组件封装

---

## 相关文件

- [vue-best-practices.md](./vue-best-practices.md) - Vue 3 开发最佳实践
- [store-bc.md](./store-bc.md) - Pinia 状态管理
- [ui-mobile.md](./ui-mobile.md) - 移动端开发规范（UniApp + UView Plus）

---

*基于 bell-plus 前端实际技术栈（antdv-next + vxe-table + vben-admin 封装）*
*RuoYi-Vue-Plus 6.X AI 开发助手*
