<template>
<el-tabs v-model="state.panel" class="ai-form-tabs">
<el-tab-pane name="designer" label="表单设计器">
  <AiFormDesigner :custom="state.custom" :show-header="true">
    <template #right="{activeData}">
      <PropPanel :active-data="activeData"></PropPanel>
    </template>
  </AiFormDesigner>
</el-tab-pane>

<el-tab-pane name="parser" label="表单解析器">
  <div class="parser-container" style="padding: 20px;">
    <el-card header="表单解析示例">
      <AiFormParser :conf="state.demoForm" ref="parserRef" />
      <div style="margin-top: 20px; text-align: center;">
        <el-button type="primary" @click="submitForm">提交表单</el-button>
      </div>
    </el-card>
  </div>
</el-tab-pane>
</el-tabs>
</template>
<script setup lang="ts">
import { reactive, provide, ref } from "vue";
import PropPanel from './demo/AiSignDemo/prop.vue'

const parserRef = ref()

const state = reactive({
  panel: 'designer',
  custom: [{
    label: "我的组件",
    name: "my",
    coms: [{
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
    }],
  }],
  demoForm: {
    formConf: {
      formRef: 'elForm',
      title: '示例表单',
      size: 'default',
      labelPosition: 'right',
      labelWidth: 100,
      gutter: 15,
      disabled: false,
      span: 24,
      formBtns: false
    },
    model: {
      name: '',
      age: null,
      gender: ''
    },
    data: [{
      "type": "default",
      "tag": "row",
      "gutter": 15,
      "children": [{
        "label": "姓名",
        "vModel": "name",
        "dataType": "string",
        "showLabel": true,
        "tag": "el-input",
        "tagIcon": "input",
        "span": 12,
        "labelWidth": null,
        "style": { "width": "100%" },
        "clearable": true,
        "readonly": false,
        "disabled": false,
        "required": true,
        "placeholder": "请输入姓名"
      }, {
        "label": "年龄",
        "vModel": "age",
        "dataType": "number",
        "showLabel": true,
        "tag": "el-input-number",
        "tagIcon": "input",
        "span": 12,
        "labelWidth": null,
        "style": { "width": "100%" },
        "clearable": true,
        "readonly": false,
        "disabled": false,
        "required": false,
        "min": 0,
        "max": 150
      }]
    }, {
      "type": "default",
      "tag": "row",
      "gutter": 15,
      "children": [{
        "label": "性别",
        "vModel": "gender",
        "dataType": "string",
        "showLabel": true,
        "tag": "el-select",
        "tagIcon": "select",
        "span": 12,
        "labelWidth": null,
        "style": { "width": "100%" },
        "clearable": true,
        "readonly": false,
        "disabled": false,
        "required": false,
        "placeholder": "请选择性别",
        "options": [{
          "label": "男",
          "value": "male"
        }, {
          "label": "女",
          "value": "female"
        }]
      }, {
        "label": "自我介绍",
        "vModel": "intro",
        "dataType": "string",
        "showLabel": true,
        "tag": "el-input",
        "type": "textarea",
        "rows": 3,
        "tagIcon": "textarea",
        "span": 24,
        "labelWidth": null,
        "style": { "width": "100%" },
        "clearable": true,
        "readonly": false,
        "disabled": false,
        "required": false,
        "placeholder": "请输入自我介绍"
      }]
    }]
  }
})

const getToken = () => {
  return 'getToken'
}
provide('getToken', getToken)

const submitForm = () => {
  parserRef.value?.submitForm((data) => {
    console.log('表单数据:', data)
  })
}
</script>
<style scoped>
.ai-form-tabs {
  height: 100vh;
}
:deep(.el-tabs__content) {
  height: calc(100% - 60px);
  overflow: auto;
}
</style>
