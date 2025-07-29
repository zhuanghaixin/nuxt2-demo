<template>
    <div class="container">
        <a-row justify="center" align="middle" style="min-height: 100vh">
            <a-col :span="16">
                <div class="hero">
                    <a-typography-title :level="1" style="text-align: center; color: #00c58e">
                        🚀 Nuxt2 Demo 项目
                    </a-typography-title>

                    <a-typography-paragraph style="text-align: center; font-size: 16px; color: #666">
                        基于 Nuxt2 + Composition API + Ant Design Vue 构建
                    </a-typography-paragraph>

                    <a-divider />

                    <a-row :gutter="[24, 24]">
                        <a-col :span="12">
                            <a-card title="✨ 技术栈" hoverable>
                                <a-tag color="green">Nuxt.js ^2.17.1</a-tag>
                                <a-tag color="blue">Vue ^2.7.14</a-tag>
                                <a-tag color="purple">@nuxtjs/composition-api ^0.34.0</a-tag>
                                <a-tag color="orange">Ant Design Vue ^1.7.8</a-tag>
                            </a-card>
                        </a-col>

                        <a-col :span="12">
                            <a-card title="🎯 功能展示" hoverable>
                                <p><strong>计数器：</strong> {{ count }}</p>
                                <a-space>
                                    <a-button type="primary" @click="increment">
                                        <a-icon type="plus" />
                                        增加
                                    </a-button>
                                    <a-button @click="decrement">
                                        <a-icon type="minus" />
                                        减少
                                    </a-button>
                                    <a-button type="dashed" @click="reset">
                                        <a-icon type="reload" />
                                        重置
                                    </a-button>
                                </a-space>
                            </a-card>
                        </a-col>
                    </a-row>

                    <a-divider />

                    <a-card title="📝 表单示例" hoverable>
                        <a-form layout="vertical">
                            <a-row :gutter="16">
                                <a-col :span="12">
                                    <a-form-item label="用户名">
                                        <a-input
                                            v-model="form.username"
                                            placeholder="请输入用户名"
                                            prefix-icon="user"
                                        />
                                    </a-form-item>
                                </a-col>
                                <a-col :span="12">
                                    <a-form-item label="邮箱">
                                        <a-input v-model="form.email" placeholder="请输入邮箱" prefix-icon="mail" />
                                    </a-form-item>
                                </a-col>
                            </a-row>
                            <a-form-item>
                                <a-button type="primary" @click="handleSubmit">
                                    <a-icon type="save" />
                                    提交表单
                                </a-button>
                            </a-form-item>
                        </a-form>
                    </a-card>

                    <a-divider />

                    <a-card title="🎯 演示页面" hoverable>
                        <a-row :gutter="16">
                            <a-col :span="12">
                                <a-card
                                    size="small"
                                    hoverable
                                    @click="$router.push('/highlight-demo')"
                                    style="cursor: pointer; height: 120px"
                                >
                                    <div style="text-align: center">
                                        <a-icon
                                            type="highlight"
                                            style="font-size: 32px; color: #1890ff; margin-bottom: 8px"
                                        />
                                        <h3>高亮文本输入组件</h3>
                                        <p style="color: #666; margin: 0">智能解析中括号，支持高亮区域编辑</p>
                                    </div>
                                </a-card>
                            </a-col>
                            <a-col :span="12">
                                <a-card size="small" hoverable style="height: 120px; opacity: 0.6">
                                    <div style="text-align: center">
                                        <a-icon type="plus" style="font-size: 32px; color: #ccc; margin-bottom: 8px" />
                                        <h3 style="color: #ccc">更多组件</h3>
                                        <p style="color: #999; margin: 0">敬请期待...</p>
                                    </div>
                                </a-card>
                            </a-col>
                        </a-row>
                    </a-card>

                    <a-divider />

                    <div style="text-align: center">
                        <a-typography-text type="secondary"> 🎉 项目已成功搭建并运行！ </a-typography-text>
                    </div>
                </div>
            </a-col>
        </a-row>
    </div>
</template>

<script>
import { defineComponent, ref, reactive } from '@nuxtjs/composition-api';

export default defineComponent({
    name: 'IndexPage',
    setup() {
        // 使用 Composition API 的响应式数据
        const count = ref(0);

        const form = reactive({
            username: '',
            email: '',
        });

        // 方法
        const increment = () => {
            count.value++;
        };

        const decrement = () => {
            count.value--;
        };

        const reset = () => {
            count.value = 0;
        };

        const handleSubmit = () => {
            if (!form.username || !form.email) {
                // 使用 Ant Design Vue 的 message 组件
                return;
            }

            console.log('表单数据：', form);
            // 可以在这里添加提交逻辑
        };

        return {
            count,
            form,
            increment,
            decrement,
            reset,
            handleSubmit,
        };
    },
});
</script>

<style scoped>
.container {
    padding: 20px;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    min-height: 100vh;
}

.hero {
    background: white;
    border-radius: 12px;
    padding: 40px;
    box-shadow: 0 10px 30px rgba(0, 0, 0, 0.1);
}

.ant-card {
    border-radius: 8px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.06);
}

.ant-tag {
    margin: 4px;
}
</style>
