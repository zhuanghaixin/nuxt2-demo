<!-- 高亮文本输入框组件的主模板 -->
<template>
    <!-- 主容器：监听点击事件处理空白区域点击聚焦到最后一个块 -->
    <div class="highlight-input" ref="containerRef" @click="handleContainerClick" :class="{ 'is-empty': isInputEmpty }">
        <!--
      动态渲染任意数量的文本块和高亮块
      每个span元素代表一个可编辑的文本块，可以是：
      1. 普通文本块：显示普通文本，用户可以编辑
      2. 高亮块：显示高亮文本，有占位符功能
    -->
        <span
            v-for="(block, index) in blocks"
            :key="block.id"
            :class="[
                block.type === 'text' ? 'normal-text' : 'highlight-block',
                { 'has-content': block.type === 'highlight' && block.hasUserInput },
                { 'after-highlight': index > 0 && blocks[index - 1]?.type === 'highlight' },
            ]"
            contenteditable="true"
            :data-index="index"
            :data-placeholder="block.type === 'highlight' ? block.placeholder : undefined"
            @input="onBlockInput($event, index)"
            @focus="onBlockFocus(index)"
            @blur="onBlockBlur(index)"
            @keydown="handleKeyNavigation($event, index)"
            :ref="(el) => setBlockRef(index, el)"
            spellcheck="false"
            >{{ block.type === 'highlight' ? block.displayValue : block.value }}</span
        >

        <!-- 整体占位符：当输入框为空时显示 -->
        <div v-if="isInputEmpty" class="input-placeholder" @click="handlePlaceholderClick">
            {{ placeholder }}
        </div>
    </div>
</template>

<!-- Vue3 Composition API 脚本部分 -->
<script>
// 导入Vue的响应式API和生命周期钩子 - 适配 Nuxt2 + Composition API
import { ref, onMounted, nextTick, watch, computed, defineComponent } from '@nuxtjs/composition-api';

/**
 * Nuxt2 + Composition API 高亮文本输入框组件
 * 功能：支持高亮显示特定文本区域，可编辑文本内容
 * 特点：动态解析后端返回的文本，自动识别中括号内的高亮区域
 */

export default defineComponent({
    name: 'HighlightTextInput',

    // ==================== 事件定义 ====================
    // 定义组件可以发射的事件：提交、回车、内容变化、v-model更新
    emits: ['submit', 'enter', 'change', 'update:modelValue'],

    // ==================== Props定义 ====================
    // 定义组件接收的属性
    props: {
        // v-model绑定的值（用户的实际数据）
        modelValue: {
            type: String,
            default: '',
        },
        // 组件内部的预设模板（包含中括号的模板文本）
        presetText: {
            type: String,
            default: '',
        },
        // 是否在最终输出中保留中括号
        keepBrackets: {
            type: Boolean,
            default: false,
        },
        // 是否允许回车换行
        allowLineBreak: {
            type: Boolean,
            default: false,
        },
        // 整个输入框的占位符文本（当没有任何内容时显示）
        placeholder: {
            type: String,
            default: '请输入内容...',
        },
        // Ctrl/Shift+Enter的行为：'line-break'表示换行，'submit'表示提交
        ctrlEnterAction: {
            type: String,
            default: 'line-break',
            validator: (value) => ['line-break', 'submit'].includes(value),
        },
        // 是否启用高亮块复制功能（回车时复制高亮块到下一行）
        enableHighlightDuplication: {
            type: Boolean,
            default: true,
        },
    },

    setup(props, { emit }) {
        // ==================== 响应式数据定义 ====================
        const blocks = ref([]);
        const blockRefs = ref([]);
        const containerRef = ref(null);

        // ==================== 计算属性 ====================
        const isInputEmpty = computed(() => {
            if (blocks.value.length === 0) return true;

            return blocks.value.every((block) => {
                if (block.type === 'text') {
                    return !block.value || block.value.trim() === '';
                } else {
                    return !block.hasUserInput || !block.value || block.value.trim() === '';
                }
            });
        });

        // ==================== 工具函数 ====================
        function parsePresetText(text) {
            const blocks = [];
            const regex = /\[([^\]]*)\]/g;
            let lastIndex = 0;
            let match;
            let blockId = 0;

            while ((match = regex.exec(text)) !== null) {
                if (match.index > lastIndex) {
                    const textContent = text.substring(lastIndex, match.index);
                    if (textContent) {
                        blocks.push({
                            id: `block-${blockId++}`,
                            type: 'text',
                            value: textContent,
                        });
                    }
                }

                blocks.push({
                    id: `block-${blockId++}`,
                    type: 'highlight',
                    placeholder: match[1],
                    value: '',
                    displayValue: match[1],
                    hasUserInput: false,
                });

                lastIndex = regex.lastIndex;
            }

            if (lastIndex < text.length) {
                const textContent = text.substring(lastIndex);
                if (textContent) {
                    blocks.push({
                        id: `block-${blockId++}`,
                        type: 'text',
                        value: textContent,
                    });
                }
            }

            return blocks;
        }

        function setCursorToPos(el, pos = 'start') {
            if (!el) return;

            try {
                const sel = window.getSelection();
                if (!sel) return;

                sel.removeAllRanges();
                const range = document.createRange();

                if (pos === 'start') {
                    if (el.childNodes.length > 0) {
                        const firstNode = el.childNodes[0];
                        if (firstNode.nodeType === Node.TEXT_NODE) {
                            range.setStart(firstNode, 0);
                            range.setEnd(firstNode, 0);
                        } else {
                            range.setStart(el, 0);
                            range.setEnd(el, 0);
                        }
                    } else {
                        range.setStart(el, 0);
                        range.setEnd(el, 0);
                    }
                } else {
                    if (el.childNodes.length > 0) {
                        const lastNode = el.childNodes[el.childNodes.length - 1];
                        if (lastNode.nodeType === Node.TEXT_NODE) {
                            const textLength = lastNode.textContent.length;
                            range.setStart(lastNode, textLength);
                            range.setEnd(lastNode, textLength);
                        } else {
                            range.setStart(el, el.childNodes.length);
                            range.setEnd(el, el.childNodes.length);
                        }
                    } else {
                        const textLength = el.textContent.length;
                        range.setStart(el, textLength);
                        range.setEnd(el, textLength);
                    }
                }

                sel.addRange(range);
            } catch (error) {
                // 静默处理错误
            }
        }

        // ==================== 事件处理函数 ====================
        function setBlockRef(index, el) {
            blockRefs.value[index] = el;
        }

        function onBlockInput(e, index) {
            const value = e.target.innerText.replace(/\n/g, '');
            const block = blocks.value[index];

            if (!block) return;

            if (block.type === 'text') {
                block.value = value;
            } else {
                block.value = value;
                block.hasUserInput = value.length > 0;
                block.displayValue = value;
            }

            emitChangeEvent();
        }

        function onBlockFocus(index) {
            const block = blocks.value[index];
            if (!block) return;

            if (block.type === 'highlight') {
                if (!block.hasUserInput) {
                    nextTick(() => {
                        const el = blockRefs.value[index];
                        if (el) {
                            const sel = window.getSelection();
                            const range = document.createRange();
                            if (sel) {
                                range.selectNodeContents(el);
                                sel.removeAllRanges();
                                sel.addRange(range);
                            }
                        }
                    });
                }
            }
        }

        function onBlockBlur(index) {
            const block = blocks.value[index];
            if (!block) return;

            if (block.type === 'highlight' && !block.hasUserInput) {
                block.displayValue = block.placeholder;
            }
        }

        function handleKeyNavigation(e, currentIndex) {
            const isLeft = e.key === 'ArrowLeft';
            const isRight = e.key === 'ArrowRight';
            const isEnter = e.key === 'Enter';

            if (isEnter && !props.allowLineBreak) {
                e.preventDefault();
                if (currentIndex < blocks.value.length - 1) {
                    focusBlock(currentIndex + 1, 'start');
                } else {
                    handleFormSubmit();
                }
                return;
            }

            if (!isLeft && !isRight) return;

            const el = e.target;
            const sel = window.getSelection();
            if (!sel || sel.rangeCount === 0) return;

            const range = sel.getRangeAt(0);
            const isAtStart = range.startOffset === 0;
            const isAtEnd = range.startOffset === el.textContent.length;

            if (isLeft && isAtStart && currentIndex > 0) {
                e.preventDefault();
                focusBlock(currentIndex - 1, 'end');
            } else if (isRight && isAtEnd && currentIndex < blocks.value.length - 1) {
                e.preventDefault();
                focusBlock(currentIndex + 1, 'start');
            }
        }

        function handleContainerClick(e) {
            if (e.target === e.currentTarget) {
                e.preventDefault();
                const lastIndex = blocks.value.length - 1;
                if (lastIndex >= 0) {
                    nextTick(() => {
                        focusBlock(lastIndex, 'end');
                    });
                }
            }
        }

        function handlePlaceholderClick() {
            if (blocks.value.length === 0) {
                initializeComponent();
            }

            nextTick(() => {
                const firstHighlightIndex = blocks.value.findIndex((block) => block.type === 'highlight');
                if (firstHighlightIndex !== -1) {
                    focusBlock(firstHighlightIndex, 'start');
                } else if (blocks.value.length > 0) {
                    focusBlock(0, 'start');
                }
            });
        }

        function focusBlock(index, pos) {
            setTimeout(() => {
                const targetRef = blockRefs.value[index];
                if (targetRef && document.contains(targetRef)) {
                    targetRef.focus();
                    nextTick(() => {
                        if (pos === 'start') {
                            setCursorToPos(targetRef, 'start');
                        } else {
                            setCursorToPos(targetRef, 'end');
                        }
                    });
                }
            }, 10);
        }

        function getFullText() {
            let result = '';
            blocks.value.forEach((block) => {
                if (block.type === 'text') {
                    result += block.value;
                } else {
                    const content = block.hasUserInput ? block.value : block.placeholder;
                    if (props.keepBrackets) {
                        result += `[${content}]`;
                    } else {
                        result += content;
                    }
                }
            });
            return result;
        }

        function emitChangeEvent() {
            const fullText = getFullText();
            emit('update:modelValue', fullText);

            const textBlocks = [];
            const highlightBlocks = [];

            blocks.value.forEach((block, index) => {
                if (block.type === 'text') {
                    textBlocks.push({ index, value: block.value });
                } else {
                    highlightBlocks.push({
                        index,
                        value: block.value,
                        placeholder: block.placeholder,
                        hasUserInput: block.hasUserInput,
                    });
                }
            });

            emit('change', {
                fullText,
                textBlocks,
                highlightBlocks,
                totalBlocks: blocks.value.length,
            });
        }

        function handleFormSubmit() {
            const fullText = getFullText();
            const textBlocks = [];
            const highlightBlocks = [];

            blocks.value.forEach((block, index) => {
                if (block.type === 'text') {
                    textBlocks.push({ index, value: block.value });
                } else {
                    highlightBlocks.push({
                        index,
                        value: block.value,
                        placeholder: block.placeholder,
                        hasUserInput: block.hasUserInput,
                    });
                }
            });

            emit('submit', {
                fullText,
                textBlocks,
                highlightBlocks,
                totalBlocks: blocks.value.length,
            });

            emit('enter', fullText);
        }

        function initializeComponent() {
            const currentUserInputs = new Map();
            if (blocks.value.length > 0) {
                blocks.value.forEach((block, index) => {
                    if (block.type === 'highlight' && block.hasUserInput) {
                        currentUserInputs.set(index, {
                            value: block.value,
                            hasUserInput: block.hasUserInput,
                        });
                    }
                });
            }

            if (!props.modelValue && props.presetText) {
                emit('update:modelValue', props.presetText);
                blocks.value = parsePresetText(props.presetText);
            } else {
                const textToUse = props.modelValue || props.presetText;
                blocks.value = parsePresetText(textToUse);
            }

            if (currentUserInputs.size > 0) {
                blocks.value.forEach((block, index) => {
                    if (block.type === 'highlight' && currentUserInputs.has(index)) {
                        const savedInput = currentUserInputs.get(index);
                        block.value = savedInput.value;
                        block.hasUserInput = savedInput.hasUserInput;
                        block.displayValue = savedInput.value;
                    }
                });
            }

            blockRefs.value = [];
        }

        // ==================== 生命周期钩子 ====================
        onMounted(() => {
            initializeComponent();
            nextTick(() => {
                nextTick(() => {
                    const firstHighlightIndex = blocks.value.findIndex((block) => block.type === 'highlight');
                    if (firstHighlightIndex !== -1) {
                        focusBlock(firstHighlightIndex, 'start');
                    }
                });
            });
        });

        // ==================== 响应式监听器 ====================
        watch(
            () => props.modelValue,
            (newValue) => {
                if (newValue !== undefined) {
                    const currentText = getFullText();
                    if (newValue !== currentText) {
                        initializeComponent();
                    }
                }
            },
            { immediate: false }
        );

        watch(
            () => props.presetText,
            (newPresetText) => {
                if (newPresetText) {
                    const hasUserInput = blocks.value.some((block) => block.type === 'highlight' && block.hasUserInput);

                    if (!props.modelValue && !hasUserInput) {
                        emit('update:modelValue', newPresetText);
                    }

                    initializeComponent();
                }
            },
            { immediate: false }
        );

        return {
            blocks,
            blockRefs,
            containerRef,
            isInputEmpty,
            setBlockRef,
            onBlockInput,
            onBlockFocus,
            onBlockBlur,
            handleKeyNavigation,
            handleContainerClick,
            handlePlaceholderClick,
        };
    },
});
</script>

<style scoped>
/* ==================== 主容器样式 ==================== */
.highlight-input {
    width: 100%;
    min-height: 40px;
    border: 1px solid #dcdfe6;
    border-radius: 6px;
    padding: 8px 12px;
    background: #fff;
    font-size: 16px;
    line-height: 1.8;
    cursor: text;
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
    text-align: left;
    word-break: break-all;
    word-wrap: break-word;
    overflow-wrap: break-word;
    position: relative;
    white-space: pre-wrap;
    box-sizing: border-box;
}

.highlight-input::after {
    content: '';
    flex: 1;
    min-height: 1px;
}

.highlight-input:hover {
    border-color: #c0c4cc;
}

.highlight-input:focus-within {
    border-color: #409eff;
    box-shadow: 0 0 0 2px rgba(64, 158, 255, 0.2);
}

/* ==================== 普通文本块样式 ==================== */
.normal-text[contenteditable] {
    outline: none;
    color: #333;
    background: transparent;
    margin: 0;
    border: none;
    white-space: pre-wrap;
    word-break: break-all;
    word-wrap: break-word;
    overflow-wrap: break-word;
    hyphens: auto;
    text-align: left;
    display: inline;
    vertical-align: baseline;
    caret-color: #333;
    min-width: 1px;
    min-height: 1.2em;
}

.normal-text[contenteditable]:empty:before {
    content: '\200B';
}

/* ==================== 整体占位符样式 ==================== */
.input-placeholder {
    position: absolute;
    top: 8px;
    left: 12px;
    right: 12px;
    color: #999;
    font-size: 16px;
    line-height: 1.8;
    pointer-events: auto;
    cursor: text;
    user-select: none;
    z-index: 1;
}

.highlight-input:not(.is-empty) .input-placeholder {
    display: none;
}

/* ==================== 高亮区域块样式 ==================== */
.highlight-block[contenteditable] {
    outline: none;
    min-width: 2em;
    min-height: 1.2em;
    margin: 0;
    padding: 2px 6px;
    border-radius: 4px;
    background: #f0f9ff;
    color: #409eff;
    font-weight: 500;
    position: relative;
    border: none;
    white-space: pre-wrap;
    word-break: break-all;
    word-wrap: break-word;
    overflow-wrap: break-word;
    hyphens: auto;
    text-align: left;
    display: inline-block;
    vertical-align: top;
    caret-color: #409eff;
    max-width: 100%;
    box-sizing: border-box;
}

.highlight-block[contenteditable]:focus {
    background: #e6f7ff;
}

/* ==================== 用户输入状态样式 ==================== */
.highlight-block.has-content {
    color: #1760a0;
    font-weight: 700;
    background: #f0f9ff;
}

.highlight-block.has-content:focus {
    color: #1760a0;
    background: #e6f7ff;
}

/* ==================== 占位符样式 ==================== */
.highlight-block[contenteditable]:empty:before {
    content: attr(data-placeholder);
    pointer-events: none;
}

.highlight-block[contenteditable]:focus:empty:before {
    opacity: 0.3;
}

/* ==================== 响应式设计 ==================== */
@media (max-width: 768px) {
    .highlight-input {
        font-size: 14px;
        padding: 6px 10px;
    }

    .highlight-block[contenteditable] {
        padding: 1px 4px;
        min-width: 1.5em;
    }
}
</style>
