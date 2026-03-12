<template>
  <el-tabs v-model="state.activeTab" type="border-card" style="height: 100%;">
    <el-tab-pane label="表单设计器" name="designer">
      <AiFormDesigner
        ref="designerRef"
        :custom="state.custom"
        :show-header="true"
        @change="onDesignerChange"
      >
        <template #right="{ activeData }">
          <PropPanel :active-data="activeData"></PropPanel>
        </template>
      </AiFormDesigner>
    </el-tab-pane>
    <el-tab-pane label="表单解析器" name="parser">
      <div class="parser-container">
        <el-row :gutter="20" style="height: 100%">
          <el-col :span="12" style="height: 100%">
            <div class="json-editor">
              <div class="editor-header">
                <span>JSON Schema</span>
                <el-button-group>
                  <el-button size="small" @click="formatJson">格式化</el-button>
                  <el-button size="small" type="primary" @click="parseForm">解析表单</el-button>
                  <el-button size="small" type="success" @click="loadFromDesigner">从设计器加载</el-button>
                </el-button-group>
              </div>
              <el-input
                v-model="state.jsonSchema"
                type="textarea"
                :rows="25"
                placeholder="请输入 JSON Schema 或从设计器加载..."
                style="font-family: monospace"
              />
            </div>
          </el-col>
          <el-col :span="12" style="height: 100%">
            <div class="form-preview">
              <div class="preview-header">
                <span>表单预览</span>
                <el-button size="small" type="primary" @click="submitParsedForm">提交表单</el-button>
              </div>
              <div class="preview-content">
                <AiFormParser
                  v-if="state.parsedConf.data && state.parsedConf.data.length > 0"
                  ref="parserRef"
                  :conf="state.parsedConf"
                />
                <el-empty v-else description="请输入 JSON Schema 并点击解析" />
              </div>
            </div>
          </el-col>
        </el-row>
      </div>
    </el-tab-pane>
  </el-tabs>
</template>

<script setup lang="ts">
import { reactive, ref, provide } from "vue";
import { ElMessage } from "element-plus";
import PropPanel from "./demo/AiSignDemo/prop.vue";

const designerRef = ref();
const parserRef = ref();

const state = reactive({
  activeTab: "designer",
  jsonSchema: "",
  designerData: [] as any[],
  designerFormConf: {} as any,
  parsedConf: {
    data: [] as any[],
    model: {} as any,
    formConf: {
      size: "default",
      labelPosition: "right",
      labelWidth: 100,
      gutter: 15,
      disabled: false,
      span: 24,
    },
  },
  custom: [
    {
      label: "我的组件",
      name: "my",
      coms: [
        {
          title: "扩展组件",
          model: [
            {
              label: "编辑电子签名",
              dataType: "string",
              showLabel: true,
              tag: "AiSignDemo",
              tagIcon: "input",
              span: 24,
              labelWidth: null,
              style: { width: "100%" },
              clearable: true,
              prepend: "",
              append: "",
              readonly: false,
              disabled: false,
              isCrop: true,
              required: true,
            },
          ],
          color: "var(--el-color-primary)",
        },
      ],
    },
  ],
});

const getToken = () => {
  console.log("getToken");
  return "getToken";
};

provide("getToken", getToken);

const onDesignerChange = (data: any[]) => {
  state.designerData = data;
};

const formatJson = () => {
  try {
    const parsed = JSON.parse(state.jsonSchema);
    state.jsonSchema = JSON.stringify(parsed, null, 2);
  } catch (e) {
    ElMessage.error("JSON 格式不正确");
  }
};

const parseForm = () => {
  try {
    const schema = JSON.parse(state.jsonSchema);
    state.parsedConf = {
      data: schema.data || schema.formItems || [],
      model: schema.model || {},
      formConf: {
        size: schema.formConf?.size || "default",
        labelPosition: schema.formConf?.labelPosition || "right",
        labelWidth: schema.formConf?.labelWidth || 100,
        gutter: schema.formConf?.gutter || 15,
        disabled: schema.formConf?.disabled || false,
        span: schema.formConf?.span || 24,
        title: schema.formConf?.title || "",
        model: schema.model || {},
      },
    };
    ElMessage.success("表单解析成功");
  } catch (e) {
    ElMessage.error("JSON 格式不正确，请检查");
  }
};

const loadFromDesigner = () => {
  if (state.designerData && state.designerData.length > 0) {
    const schema = {
      data: state.designerData,
      model: {},
      formConf: {
        size: "default",
        labelPosition: "right",
        labelWidth: 100,
        gutter: 15,
        disabled: false,
        span: 24,
      },
    };
    state.jsonSchema = JSON.stringify(schema, null, 2);
    ElMessage.success("已从设计器加载表单数据");
  } else {
    ElMessage.warning("设计器中没有表单数据，请先设计表单");
  }
};

const submitParsedForm = () => {
  if (parserRef.value) {
    parserRef.value.submitForm((model: any) => {
      console.log("表单数据:", model);
      ElMessage.success("表单提交成功，数据已打印到控制台");
    });
  }
};
</script>

<style scoped>
.parser-container {
  height: calc(100vh - 60px);
  padding: 10px;
}

.json-editor,
.form-preview {
  height: 100%;
  display: flex;
  flex-direction: column;
  border: 1px solid #e4e7ed;
  border-radius: 4px;
}

.editor-header,
.preview-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px 15px;
  background: #f5f7fa;
  border-bottom: 1px solid #e4e7ed;
  font-weight: bold;
}

.json-editor :deep(.el-textarea__inner) {
  border: none;
  border-radius: 0;
  height: calc(100% - 50px);
}

.preview-content {
  flex: 1;
  padding: 20px;
  overflow: auto;
}

.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}
.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}
.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}
</style>
