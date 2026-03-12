<template>
  <component v-if="!!components[item.tag]&&!item.disTag" :is="components[item.tag]" :conf="conf" :item="item" v-bind="$attrs" :form-data="modelValue" v-model="modelValue[item.vModel]">
    <template v-for="(val, name) in item.__slot__" v-slot:[name]>
      {{ val }}
    </template>
  </component>
  <component v-else :is="item.disTag??item.tag" v-bind="$attrs" :form-data="modelValue" v-model="modelValue[item.vModel]">
    <template v-for="(val, name) in item.__slot__" v-slot:[name]>
      {{ val }}
    </template>
  </component>
</template>

<script lang="ts" setup name="render-component">
import { defineAsyncComponent, computed, ref, watch, inject, reactive, h, toRefs } from 'vue'

let components = {}

const elModules = import.meta.globEager('./Elements/FormItems/My*/index.vue')
for (const path in elModules) {
  let cname = elModules[path].default.name
  components[cname] = elModules[path].default
}

const props = defineProps({
  item: Object,
  attrs: Object,
  conf: Object,
})

// 使用计算属性来安全地访问 model
const modelValue = computed(() => {
  return props.conf?.model || {}
})

// 初始化默认值
if (props.conf?.model && props.item?.vModel && props.conf.model[props.item.vModel] === undefined) {
  props.conf.model[props.item.vModel] = props.item.defaultValue
}
</script>