<template>
    <div class="container">
        <a-row justify="center" style="min-height: 100vh; padding: 20px">
            <a-col :span="20">
                <div class="demo-content">
                    <!-- 页面标题 -->
                    <div class="page-header">
                        <a-button
                            type="link"
                            icon="arrow-left"
                            size="large"
                            @click="$router.push('/')"
                            style="margin-bottom: 16px"
                        >
                            返回首页
                        </a-button>

                        <a-typography-title :level="1" style="text-align: center; color: #1890ff">
                            🎯 高亮文本输入组件演示
                        </a-typography-title>

                        <a-typography-paragraph
                            style="text-align: center; font-size: 16px; color: #666; margin-bottom: 40px"
                        >
                            智能解析中括号，支持高亮区域编辑，键盘导航，适用于法律文书、合同模板等场景
                        </a-typography-paragraph>
                    </div>

                    <!-- 基础演示 -->
                    <a-card title="📝 基础使用演示" style="margin-bottom: 24px" hoverable>
                        <template #extra>
                            <a-tag color="blue">基础功能</a-tag>
                        </template>

                        <a-form layout="vertical">
                            <a-form-item label="模板文本">
                                <a-input
                                    v-model="basicTemplate"
                                    placeholder="请输入包含[占位符]的模板文本"
                                    @pressEnter="updateBasicDemo"
                                />
                                <div style="margin-top: 8px">
                                    <a-button size="small" @click="useTemplate('legal')">法律文书模板</a-button>
                                    <a-button size="small" @click="useTemplate('contract')" style="margin-left: 8px"
                                        >合同模板</a-button
                                    >
                                    <a-button size="small" @click="useTemplate('notice')" style="margin-left: 8px"
                                        >通知模板</a-button
                                    >
                                </div>
                            </a-form-item>

                            <a-form-item label="高亮文本输入组件">
                                <HighlightTextInput
                                    v-model="basicContent"
                                    :preset-text="basicTemplate"
                                    placeholder="请点击高亮区域开始输入..."
                                    @change="handleBasicChange"
                                    @submit="handleBasicSubmit"
                                />
                            </a-form-item>

                            <a-form-item label="输出结果">
                                <a-textarea
                                    :value="basicContent"
                                    :rows="3"
                                    readonly
                                    placeholder="在上方输入组件中编辑内容，这里会实时显示结果"
                                />
                            </a-form-item>
                        </a-form>
                    </a-card>

                    <!-- 高级配置演示 -->
                    <a-card title="⚙️ 高级配置演示" style="margin-bottom: 24px" hoverable>
                        <template #extra>
                            <a-tag color="purple">高级功能</a-tag>
                        </template>

                        <a-row :gutter="24">
                            <a-col :span="12">
                                <a-form layout="vertical">
                                    <a-form-item label="配置选项">
                                        <a-space direction="vertical" style="width: 100%">
                                            <a-checkbox v-model="advancedConfig.keepBrackets">
                                                保留中括号格式
                                            </a-checkbox>
                                            <a-checkbox v-model="advancedConfig.allowLineBreak">
                                                允许回车换行
                                            </a-checkbox>
                                            <a-checkbox v-model="advancedConfig.enableHighlightDuplication">
                                                启用高亮块复制功能
                                            </a-checkbox>
                                        </a-space>
                                    </a-form-item>

                                    <a-form-item label="Ctrl+Enter行为">
                                        <a-radio-group v-model="advancedConfig.ctrlEnterAction">
                                            <a-radio-button value="line-break">换行</a-radio-button>
                                            <a-radio-button value="submit">提交</a-radio-button>
                                        </a-radio-group>
                                    </a-form-item>

                                    <a-form-item label="自定义占位符">
                                        <a-input
                                            v-model="advancedConfig.placeholder"
                                            placeholder="输入自定义占位符文本"
                                        />
                                    </a-form-item>
                                </a-form>
                            </a-col>

                            <a-col :span="12">
                                <a-form-item label="高级高亮文本输入">
                                    <HighlightTextInput
                                        v-model="advancedContent"
                                        :preset-text="advancedTemplate"
                                        :keep-brackets="advancedConfig.keepBrackets"
                                        :allow-line-break="advancedConfig.allowLineBreak"
                                        :enable-highlight-duplication="advancedConfig.enableHighlightDuplication"
                                        :ctrl-enter-action="advancedConfig.ctrlEnterAction"
                                        :placeholder="advancedConfig.placeholder"
                                        @change="handleAdvancedChange"
                                        @submit="handleAdvancedSubmit"
                                    />
                                </a-form-item>

                                <a-form-item label="详细输出信息">
                                    <a-textarea :value="formatAdvancedResult()" :rows="6" readonly />
                                </a-form-item>
                            </a-col>
                        </a-row>
                    </a-card>

                    <!-- 事件监听演示 -->
                    <a-card title="🔔 事件监听演示" hoverable>
                        <template #extra>
                            <a-tag color="green">事件处理</a-tag>
                        </template>

                        <a-row :gutter="24">
                            <a-col :span="14">
                                <HighlightTextInput
                                    v-model="eventContent"
                                    :preset-text="eventTemplate"
                                    @change="handleEventChange"
                                    @submit="handleEventSubmit"
                                    @enter="handleEventEnter"
                                />
                            </a-col>

                            <a-col :span="10">
                                <a-typography-title :level="4">事件日志</a-typography-title>
                                <div class="event-log">
                                    <div
                                        v-for="(log, index) in eventLogs"
                                        :key="index"
                                        class="event-item"
                                        :class="log.type"
                                    >
                                        <strong>{{ log.type }}:</strong> {{ log.message }}
                                        <small>{{ log.time }}</small>
                                    </div>
                                </div>
                                <a-button size="small" type="dashed" @click="clearEventLogs" style="margin-top: 8px">
                                    清空日志
                                </a-button>
                            </a-col>
                        </a-row>
                    </a-card>
                </div>
            </a-col>
        </a-row>
    </div>
</template>

<script>
import { defineComponent, ref, reactive } from '@nuxtjs/composition-api';
import HighlightTextInput from '@/components/HighlightTextInput.vue';

export default defineComponent({
    name: 'HighlightDemo',
    components: {
        HighlightTextInput,
    },

    setup() {
        // 基础演示数据
        const basicTemplate = ref('我需要[第一个字段]和[第二个字段]的帮助。');
        const basicContent = ref('');

        // 高级配置演示数据
        const advancedTemplate = ref(
            '合同编号：[合同编号]，甲方：[甲方名称]，乙方：[乙方名称]，签订日期：[签订日期]。'
        );
        const advancedContent = ref('');
        const advancedChangeData = ref(null);

        const advancedConfig = reactive({
            keepBrackets: false,
            allowLineBreak: true,
            enableHighlightDuplication: true,
            ctrlEnterAction: 'line-break',
            placeholder: '请输入合同内容...',
        });

        // 事件演示数据
        const eventTemplate = ref('请在[这里]输入您的[内容]。');
        const eventContent = ref('');
        const eventLogs = ref([]);

        // 预设模板
        const templates = {
            legal: '根据《[法律条文]》第[条款编号]条规定，[当事人]应当承担[责任类型]责任。',
            contract: '甲方：[甲方名称]，乙方：[乙方名称]，就[合作事项]达成如下协议：[协议内容]。',
            notice: '兹通知[收件人]，根据[依据文件]，定于[时间]在[地点]召开[会议名称]。',
        };

        // 方法
        function useTemplate(type) {
            basicTemplate.value = templates[type];
            updateBasicDemo();
        }

        function updateBasicDemo() {
            basicContent.value = '';
        }

        function handleBasicChange(data) {
            console.log('基础演示 - 内容变化:', data);
        }

        function handleBasicSubmit(data) {
            console.log('基础演示 - 提交事件:', data);
            addEventLog('submit', `基础演示提交: ${data.fullText}`);
        }

        function handleAdvancedChange(data) {
            advancedChangeData.value = data;
            console.log('高级演示 - 内容变化:', data);
        }

        function handleAdvancedSubmit(data) {
            console.log('高级演示 - 提交事件:', data);
            addEventLog('submit', `高级演示提交: ${data.fullText}`);
        }

        function handleEventChange(data) {
            addEventLog('change', `内容变化: ${data.fullText}`);
        }

        function handleEventSubmit(data) {
            addEventLog('submit', `表单提交: ${data.fullText}`);
        }

        function handleEventEnter(text) {
            addEventLog('enter', `回车事件: ${text}`);
        }

        function addEventLog(type, message) {
            const now = new Date();
            eventLogs.value.unshift({
                type,
                message: message.substring(0, 50) + (message.length > 50 ? '...' : ''),
                time: now.toLocaleTimeString(),
            });

            // 保持日志数量不超过10条
            if (eventLogs.value.length > 10) {
                eventLogs.value.pop();
            }
        }

        function clearEventLogs() {
            eventLogs.value = [];
        }

        function formatAdvancedResult() {
            if (!advancedChangeData.value) {
                return '在高亮文本输入组件中编辑内容，这里会显示详细的数据结构信息';
            }

            return JSON.stringify(advancedChangeData.value, null, 2);
        }

        return {
            // 基础演示
            basicTemplate,
            basicContent,
            useTemplate,
            updateBasicDemo,
            handleBasicChange,
            handleBasicSubmit,

            // 高级演示
            advancedTemplate,
            advancedContent,
            advancedConfig,
            handleAdvancedChange,
            handleAdvancedSubmit,
            formatAdvancedResult,

            // 事件演示
            eventTemplate,
            eventContent,
            eventLogs,
            handleEventChange,
            handleEventSubmit,
            handleEventEnter,
            clearEventLogs,
        };
    },
});
</script>

<style scoped>
.container {
    background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
    min-height: 100vh;
}

.demo-content {
    background: white;
    border-radius: 12px;
    padding: 32px;
    box-shadow: 0 10px 30px rgba(0, 0, 0, 0.1);
}

.page-header {
    margin-bottom: 32px;
}

.event-log {
    max-height: 300px;
    overflow-y: auto;
    border: 1px solid #f0f0f0;
    border-radius: 6px;
    padding: 12px;
    background: #fafafa;
}

.event-item {
    padding: 8px 0;
    border-bottom: 1px solid #f0f0f0;
    font-size: 13px;
}

.event-item:last-child {
    border-bottom: none;
}

.event-item.change {
    color: #1890ff;
}

.event-item.submit {
    color: #52c41a;
}

.event-item.enter {
    color: #722ed1;
}

.event-item small {
    display: block;
    color: #999;
    margin-top: 2px;
}

.ant-card {
    border-radius: 8px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.06);
}

.ant-btn + .ant-btn {
    margin-left: 8px;
}
</style>
