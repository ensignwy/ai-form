

<template> 
  <div class="app-container">
    <!-- 顶部导航 -->
    <el-header class="app-header" v-if="showHeader">
      <div class="header-left">
        <span class="logo">
          <strong style="color: var(--el-color-primary)">A</strong>
          <strong style="color: var(--el-color-danger)">i</strong>
          <strong style="color: var(--el-color-primary)">F</strong>
          <strong style="color: var(--el-color-success)">o</strong>
          <strong style="color: var(--el-color-warning)">r</strong>
          <strong style="color: var(--el-color-danger)">m</strong>
          表单设计器
        </span>
      </div>
      <div class="header-center">
        <el-radio-group v-model="currentMode" size="large">
          <el-radio-button label="designer">表单设计</el-radio-button>
          <el-radio-button label="parser">表单解析</el-radio-button>
        </el-radio-group>
      </div>
      <div class="header-right">
        <el-button type="primary" @click="showJson = true" v-if="currentMode === 'designer'">查看JSON</el-button>
      </div>
    </el-header>

    <!-- 表单设计器模式 -->
    <div v-if="currentMode === 'designer'" class="mode-container">
      <AiFormDesigner :custom="state.custom" :show-header="false" ref="designerRef">
        <template #right="{activeData}"> 
          <PropPanel :active-data="activeData"></PropPanel>
        </template>
      </AiFormDesigner>
    </div>

    <!-- 表单解析器模式 -->
    <div v-else class="mode-container parser-mode">
      <el-container style="height: 100%">
        <el-aside width="400px" class="json-panel">
          <el-tabs v-model="jsonTab" style="height: 100%">
            <el-tab-pane label="JSON Schema" name="schema">
              <div class="json-editor-wrapper">
                <el-alert
                  title="在此粘贴表单JSON Schema进行解析"
                  type="info"
                  :closable="false"
                  style="margin-bottom: 10px"
                />
                <el-input
                  v-model="jsonSchema"
                  type="textarea"
                  :rows="20"
                  placeholder="请输入表单JSON Schema..."
                  class="json-editor"
                />
                <div class="json-actions">
                  <el-button type="primary" @click="parseJsonSchema" :disabled="!jsonSchema">
                    <el-icon><View /></el-icon>
                    解析表单
                  </el-button>
                  <el-button @click="loadExample">
                    <el-icon><Document /></el-icon>
                    加载示例
                  </el-button>
                  <el-button @click="clearJson">
                    <el-icon><Delete /></el-icon>
                    清空
                  </el-button>
                </div>
              </div>
            </el-tab-pane>
            <el-tab-pane label="表单数据" name="data">
              <div class="json-editor-wrapper">
                <el-alert
                  title="表单提交的数据"
                  type="success"
                  :closable="false"
                  style="margin-bottom: 10px"
                />
                <el-input
                  v-model="formDataJson"
                  type="textarea"
                  :rows="20"
                  readonly
                  placeholder="表单数据将显示在这里..."
                  class="json-editor"
                />
              </div>
            </el-tab-pane>
          </el-tabs>
        </el-aside>
        <el-main class="parser-main">
          <el-card class="parser-card" v-if="parsedConfig.data && parsedConfig.data.length > 0">
            <template #header>
              <div class="parser-header">
                <span>{{ parsedConfig.formConf?.title || '表单预览' }}</span>
                <el-button type="primary" @click="submitForm">
                  <el-icon><Check /></el-icon>
                  提交表单
                </el-button>
              </div>
            </template>
            <AiFormParser 
              :conf="parsedConfig" 
              ref="parserRef"
              :loading="false"
            />
          </el-card>
          <el-empty v-else description="请在左侧输入JSON Schema并点击解析表单">
            <el-button type="primary" @click="loadExample">加载示例</el-button>
          </el-empty>
        </el-main>
      </el-container>
    </div>

    <!-- JSON查看对话框 -->
    <el-dialog
      v-model="showJson"
      title="当前表单JSON Schema"
      width="800px"
      destroy-on-close
    >
      <el-input
        v-model="currentJsonSchema"
        type="textarea"
        :rows="20"
        readonly
      />
      <template #footer>
        <el-button @click="showJson = false">关闭</el-button>
        <el-button type="primary" @click="copyJson">复制JSON</el-button>
      </template>
    </el-dialog>
  </div>
</template>

<script setup lang="ts"> 
import { reactive, ref, computed, provide, onMounted, watch } from "vue"; 
import { ElMessage } from 'element-plus'
import { View, Document, Delete, Check } from '@element-plus/icons-vue'
import PropPanel from './demo/AiSignDemo/prop.vue'

const designerRef = ref()
const parserRef = ref()
const currentMode = ref('designer')
const showHeader = ref(true)
const showJson = ref(false)
const jsonTab = ref('schema')
const jsonSchema = ref('')
const formDataJson = ref('')

// 示例JSON Schema
const exampleSchema = {
  "title": "示例表单",
  "size": "small",
  "labelPosition": "right",
  "labelWidth": 100,
  "gutter": 15,
  "disabled": false,
  "span": 24,
  "formBtns": true,
  "model": {},
  "data": [
    {
      "label": "姓名",
      "dataType": "string",
      "showLabel": true,
      "tag": "el-input",
      "tagIcon": "input",
      "placeholder": "请输入姓名",
      "defaultValue": "",
      "span": 24,
      "labelWidth": null,
      "style": { "width": "100%" },
      "clearable": true,
      "prepend": "",
      "append": "",
      "readonly": false,
      "disabled": false,
      "required": true,
      "regList": [],
      "__slot__": {},
      "changeTag": true,
      "proCondition": false,
      "asSummary": false,
      "formId": "1",
      "layout": "colFormItem",
      "vModel": "field1"
    },
    {
      "label": "邮箱",
      "dataType": "string",
      "showLabel": true,
      "tag": "el-input",
      "tagIcon": "input",
      "placeholder": "请输入邮箱",
      "defaultValue": "",
      "span": 24,
      "labelWidth": null,
      "style": { "width": "100%" },
      "clearable": true,
      "prepend": "",
      "append": "",
      "readonly": false,
      "disabled": false,
      "required": true,
      "regList": [
        {
          "pattern": "/^\\S+@\\S+\\.\\S+$/",
          "message": "请输入正确的邮箱格式"
        }
      ],
      "__slot__": {},
      "changeTag": true,
      "proCondition": false,
      "asSummary": false,
      "formId": "2",
      "layout": "colFormItem",
      "vModel": "field2"
    },
    {
      "label": "年龄",
      "showLabel": true,
      "tag": "el-input-number",
      "tagIcon": "number",
      "dataType": "number",
      "placeholder": "",
      "defaultValue": undefined,
      "style": { "width": null },
      "span": 24,
      "labelWidth": null,
      "min": 0,
      "max": 150,
      "step": 1,
      "step-strictly": false,
      "precision": 0,
      "controls-position": "right",
      "disabled": false,
      "required": true,
      "regList": [],
      "changeTag": true,
      "proCondition": true,
      "formId": "3",
      "layout": "colFormItem",
      "vModel": "field3"
    },
    {
      "label": "性别",
      "showLabel": true,
      "orgTag": "el-radio-group",
      "tag": "my-radio-group",
      "tagIcon": "radio",
      "defaultValue": "",
      "span": 24,
      "labelWidth": null,
      "style": {},
      "optionType": "default",
      "border": false,
      "disabled": false,
      "required": true,
      "options": [
        { "label": "男", "value": "male" },
        { "label": "女", "value": "female" }
      ],
      "regList": [],
      "changeTag": true,
      "proCondition": true,
      "formId": "4",
      "layout": "colFormItem",
      "vModel": "field4"
    },
    {
      "label": "兴趣爱好",
      "showLabel": true,
      "orgTag": "el-checkbox-group",
      "tag": "my-checkbox-group",
      "tagIcon": "checkbox",
      "dataType": "checkbox",
      "defaultValue": [],
      "span": 24,
      "labelWidth": null,
      "style": {},
      "optionType": "default",
      "border": false,
      "disabled": false,
      "required": true,
      "multiple": true,
      "options": [
        { "label": "阅读", "value": "reading" },
        { "label": "运动", "value": "sports" },
        { "label": "音乐", "value": "music" },
        { "label": "旅游", "value": "travel" }
      ],
      "regList": [],
      "changeTag": true,
      "proCondition": false,
      "asSummary": false,
      "formId": "5",
      "layout": "colFormItem",
      "vModel": "field5"
    },
    {
      "label": "出生日期",
      "showLabel": true,
      "dataType": "date",
      "tag": "el-date-picker",
      "tagIcon": "date",
      "placeholder": "请选择",
      "defaultValue": null,
      "type": "date",
      "span": 24,
      "labelWidth": null,
      "style": { "width": "100%" },
      "disabled": false,
      "clearable": true,
      "required": true,
      "format": "YYYY-MM-DD",
      "value-format": "YYYY-MM-DD",
      "readonly": false,
      "regList": [],
      "changeTag": true,
      "proCondition": false,
      "asSummary": false,
      "formId": "6",
      "layout": "colFormItem",
      "vModel": "field6"
    },
    {
      "label": "个人简介",
      "type": "textarea",
      "showLabel": true,
      "tag": "el-input",
      "tagIcon": "textarea",
      "dataType": "string",
      "placeholder": "请输入个人简介",
      "defaultValue": undefined,
      "span": 24,
      "labelWidth": null,
      "autosize": { "minRows": 4, "maxRows": 4 },
      "style": { "width": "100%" },
      "maxlength": null,
      "show-word-limit": false,
      "readonly": false,
      "disabled": false,
      "required": false,
      "regList": [],
      "changeTag": true,
      "proCondition": false,
      "asSummary": false,
      "formId": "7",
      "layout": "colFormItem",
      "vModel": "field7"
    }
  ],
  "formConf": {
    "formRef": "elForm",
    "title": "示例表单",
    "size": "small",
    "labelPosition": "right",
    "labelWidth": 100,
    "gutter": 15,
    "disabled": false,
    "span": 24,
    "formBtns": true,
    "model": {}
  }
}

const parsedConfig = ref({
  data: [],
  formConf: {},
  model: {}
})

const currentJsonSchema = ref('')

const updateJsonSchema = async () => {
  if (designerRef.value) {
    try {
      const setting = await designerRef.value.getSetting()
      if (setting) {
        currentJsonSchema.value = JSON.stringify(setting, null, 2)
      }
    } catch (error) {
      console.log('获取表单配置失败', error)
    }
  }
}

watch(currentMode, async (newMode) => {
  if (newMode === 'designer') {
    await updateJsonSchema()
  }
})

const state = reactive({
  custom: [{
    label: "我的组件",
    name: "my",
    coms: [
      {
        title: "扩展组件",
        model: [{ 
          "label": "编辑电子签名", 
          "dataType": "string", 
          "showLabel": true, 
          "tag": "AiSignDemo", 
          "tagIcon": "input",  
          "span": 24,
          "labelWidth": null, 
          "style": { "width": "100%" }, 
          "clearable": true, 
          "prepend": "", 
          "append": "",   
          "readonly": false, 
          "disabled": false, 
          "isCrop": true,
          "required": true
        }],
        color: "var(--el-color-primary)",
      },
    ],
  }],
})

const getToken = () => {
  console.log('getToken')
  return 'getToken'
}

provide('getToken', getToken)

const parseJsonSchema = () => {
  try {
    if (!jsonSchema.value.trim()) {
      ElMessage.warning('请输入JSON Schema')
      return
    }
    const parsed = JSON.parse(jsonSchema.value)
    
    // 处理不同的JSON格式
    if (parsed.data && Array.isArray(parsed.data)) {
      parsedConfig.value = {
        data: parsed.data,
        formConf: parsed.formConf || parsed,
        model: parsed.model || {}
      }
    } else if (parsed.formItems && Array.isArray(parsed.formItems)) {
      // 从设计器导出的格式
      parsedConfig.value = {
        data: parsed.formItems,
        formConf: parsed,
        model: parsed.model || {}
      }
    } else {
      ElMessage.error('JSON格式不正确，请检查')
      return
    }
    
    ElMessage.success('表单解析成功')
  } catch (error) {
    ElMessage.error('JSON解析失败: ' + error.message)
  }
}

const loadExample = () => {
  jsonSchema.value = JSON.stringify(exampleSchema, null, 2)
  parseJsonSchema()
}

const clearJson = () => {
  jsonSchema.value = ''
  parsedConfig.value = { data: [], formConf: {}, model: {} }
  formDataJson.value = ''
}

const submitForm = () => {
  if (parserRef.value) {
    parserRef.value.submitForm((data) => {
      formDataJson.value = JSON.stringify(data, null, 2)
      jsonTab.value = 'data'
      ElMessage.success('表单提交成功，数据已显示在"表单数据"标签页')
      console.log('表单提交数据:', data)
    })
  }
}

const copyJson = () => {
  navigator.clipboard.writeText(currentJsonSchema.value).then(() => {
    ElMessage.success('JSON已复制到剪贴板')
  })
}

onMounted(() => {
  // 默认加载示例
  loadExample()
})
</script>

<style scoped>
.app-container {
  height: 100vh;
  display: flex;
  flex-direction: column;
}

.app-header {
  height: 60px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  background-color: #fff;
  border-bottom: 1px solid #e4e7ed;
  padding: 0 20px;
}

.header-left .logo {
  font-size: 20px;
  color: rgba(0, 0, 0, 0.7);
}

.header-center {
  flex: 1;
  display: flex;
  justify-content: center;
}

.header-right {
  display: flex;
  gap: 10px;
}

.mode-container {
  flex: 1;
  overflow: hidden;
}

.parser-mode {
  background-color: #f5f7fa;
}

.json-panel {
  background-color: #fff;
  border-right: 1px solid #e4e7ed;
  padding: 15px;
}

.json-editor-wrapper {
  display: flex;
  flex-direction: column;
  height: calc(100% - 40px);
}

.json-editor {
  flex: 1;
}

.json-editor :deep(.el-textarea__inner) {
  font-family: 'Consolas', 'Monaco', 'Courier New', monospace;
  font-size: 13px;
  line-height: 1.5;
}

.json-actions {
  margin-top: 15px;
  display: flex;
  gap: 10px;
  flex-wrap: wrap;
}

.parser-main {
  padding: 20px;
  overflow-y: auto;
}

.parser-card {
  max-width: 800px;
  margin: 0 auto;
}

.parser-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}
</style>
