# HighlightTextInput 组件使用说明

## 📋 组件简介

HighlightTextInput 是一个专为 Nuxt2 + Composition API 环境设计的高亮文本输入组件。它能够智能解析包含中括号的模板文本，自动将中括号内的内容转换为可编辑的高亮区域，特别适用于法律文书、合同模板、通知模板等需要动态填写内容的场景。

## 🚀 快速开始

### 基本使用

```vue
<template>
  <div>
    <HighlightTextInput 
      v-model="content"
      :preset-text="template"
      placeholder="请输入内容..."
    />
    <p>输出内容：{{ content }}</p>
  </div>
</template>

<script>
import { defineComponent, ref } from '@nuxtjs/composition-api'
import HighlightTextInput from '@/components/HighlightTextInput.vue'

export default defineComponent({
  components: {
    HighlightTextInput
  },
  setup() {
    const content = ref('')
    const template = ref('我需要[第一个字段]和[第二个字段]的帮助。')
    
    return {
      content,
      template
    }
  }
})
</script>
```

### 高级配置

```vue
<template>
  <HighlightTextInput
    v-model="formData.content"
    :preset-text="template"
    :keep-brackets="false"
    :allow-line-break="true"
    :enable-highlight-duplication="true"
    placeholder="请输入您的法律文书内容..."
    ctrl-enter-action="submit"
    @submit="handleSubmit"
    @change="handleChange"
  />
</template>

<script>
import { defineComponent, ref, reactive } from '@nuxtjs/composition-api'

export default defineComponent({
  setup() {
    const formData = reactive({
      content: ''
    })

    const template = ref('请为[案件类型]准备[证据材料]。')

    function handleSubmit(data) {
      console.log('提交的数据:', data)
    }

    function handleChange(data) {
      console.log('内容变化:', data)
    }
    
    return {
      formData,
      template,
      handleSubmit,
      handleChange
    }
  }
})
</script>
```

## 📖 API 文档

### Props 属性

| 属性名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `modelValue` | String | `''` | v-model 绑定的值，组件的主要数据源 |
| `presetText` | String | `''` | 预设模板文本，包含中括号的初始模板 |
| `keepBrackets` | Boolean | `false` | 是否在最终输出中保留中括号 |
| `allowLineBreak` | Boolean | `false` | 是否允许回车换行，false时跳转到下一个块 |
| `placeholder` | String | `'请输入内容...'` | 整体占位符文本，当输入框为空时显示 |
| `ctrlEnterAction` | String | `'line-break'` | Ctrl/Shift+Enter行为：`'line-break'`换行或`'submit'`提交 |
| `enableHighlightDuplication` | Boolean | `true` | 是否启用高亮块复制功能（回车时复制高亮块到下一行） |

### Events 事件

| 事件名 | 参数 | 说明 |
|--------|------|------|
| `update:modelValue` | `(value: string)` | v-model 更新事件 |
| `change` | `(data: object)` | 内容变化事件，包含详细信息 |
| `submit` | `(data: object)` | 提交事件，在最后一个区域按回车时触发 |
| `enter` | `(text: string)` | 回车事件，向后兼容 |

### 事件数据结构

#### change / submit 事件数据

```javascript
{
  fullText: "完整的文本内容",
  textBlocks: [
    { index: 0, value: "普通文本内容" }
  ],
  highlightBlocks: [
    {
      index: 1,
      value: "用户输入的值",
      placeholder: "占位符文本",
      hasUserInput: true
    }
  ],
  totalBlocks: 3
}
```

## 🎯 使用场景

### 1. 法律文书模板

```vue
<template>
  <HighlightTextInput
    v-model="legalDocument"
    :preset-text="'根据《[法律条文]》第[条款编号]条规定，[当事人]应当承担[责任类型]责任。'"
    :keep-brackets="true"
    @submit="saveLegalDocument"
  />
</template>
```

### 2. 合同模板

```vue
<template>
  <HighlightTextInput
    v-model="contract"
    :preset-text="'甲方：[甲方名称]，乙方：[乙方名称]，就[合作事项]达成如下协议：[协议内容]。'"
    :allow-line-break="true"
    placeholder="请填写合同内容..."
  />
</template>
```

### 3. 通知模板

```vue
<template>
  <HighlightTextInput
    v-model="notice"
    :preset-text="'兹通知[收件人]，根据[依据文件]，定于[时间]在[地点]召开[会议名称]。'"
    ctrl-enter-action="submit"
    @submit="sendNotice"
  />
</template>
```

## ⌨️ 快捷键

- **方向键** (←/→): 在高亮区域间快速导航
- **Tab**: 跳转到下一个高亮区域
- **Enter**: 跳转到下一个区域或提交（根据配置）
- **Ctrl+Enter**: 执行配置的操作（换行或提交）
- **Shift+Enter**: 执行配置的操作（换行或提交）
- **Ctrl+A**: 全选所有内容
- **Ctrl+C**: 复制完整文本内容
- **Backspace**: 在高亮区域开头时清空内容，再次按下删除整个区域
- **Delete**: 在高亮区域结尾时清空内容，再次按下删除整个区域

## 🎨 样式定制

组件提供了丰富的 CSS 类，可以根据需要进行样式定制：

```css
/* 主容器 */
.highlight-input {
  /* 自定义样式 */
}

/* 普通文本块 */
.normal-text {
  /* 自定义普通文本样式 */
}

/* 高亮块 */
.highlight-block {
  /* 自定义高亮块样式 */
}

/* 有用户输入的高亮块 */
.highlight-block.has-content {
  /* 自定义有内容的高亮块样式 */
}

/* 整体占位符 */
.input-placeholder {
  /* 自定义占位符样式 */
}
```

## 🔧 高级功能

### 1. 动态模板更新

```vue
<script>
export default defineComponent({
  setup() {
    const template = ref('初始模板[字段1]')
    
    // 动态更新模板
    function updateTemplate() {
      template.value = '新的模板[字段1]和[字段2]'
    }
    
    return {
      template,
      updateTemplate
    }
  }
})
</script>
```

### 2. 批量数据处理

```vue
<script>
export default defineComponent({
  setup() {
    function handleBatchSubmit(data) {
      // 处理高亮块数据
      const processedData = data.highlightBlocks.map(block => ({
        field: block.placeholder,
        value: block.value,
        filled: block.hasUserInput
      }))
      
      console.log('批量处理结果:', processedData)
    }
    
    return {
      handleBatchSubmit
    }
  }
})
</script>
```

### 3. 自定义验证

```vue
<script>
export default defineComponent({
  setup() {
    function validateContent(data) {
      const emptyFields = data.highlightBlocks
        .filter(block => !block.hasUserInput)
        .map(block => block.placeholder)
      
      if (emptyFields.length > 0) {
        console.warn('以下字段未填写:', emptyFields)
        return false
      }
      
      return true
    }
    
    return {
      validateContent
    }
  }
})
</script>
```

## 🚨 注意事项

1. **版本兼容性**: 组件专为 Nuxt2 + @nuxtjs/composition-api 环境设计
2. **浏览器支持**: 需要支持 `contenteditable` 的现代浏览器
3. **中括号格式**: 模板文本中的中括号 `[]` 会被自动解析为高亮区域
4. **数据绑定**: 使用 v-model 进行双向数据绑定
5. **事件处理**: 建议监听 `change` 事件进行实时数据处理

## 🐛 常见问题

### Q: 为什么高亮区域没有显示？
A: 请检查 `presetText` 是否包含正确格式的中括号 `[内容]`

### Q: 如何禁用某些快捷键？
A: 可以通过 CSS 或在父组件中阻止事件冒泡来禁用特定快捷键

### Q: 能否自定义高亮区域的样式？
A: 可以，通过覆盖 `.highlight-block` 等 CSS 类来自定义样式

### Q: 组件是否支持多行文本？
A: 支持，将 `allowLineBreak` 设置为 `true` 即可启用多行编辑

## 🔄 更新日志

- **v1.0.0**: 初始版本，支持基础高亮文本编辑功能
- 适配 Nuxt2 + Composition API 环境
- 完整的键盘导航支持
- 丰富的配置选项和事件回调 