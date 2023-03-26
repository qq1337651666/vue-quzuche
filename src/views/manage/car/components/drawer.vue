<template>
  <el-drawer
      :title="我是标题"
      model-value="drawerVisible"
      :with-header="false">
    <span style="font-size: 30px">{{ remarkTitle }}</span>
      <el-skeleton />
    <span style="font-size: 16px">{{props.remarkContent}}</span>
  </el-drawer>
</template>
<script setup>
import { defineProps, defineEmits } from "vue";
//vue3 defineProps 父-》子  defineEmits-》子-》父
 const props = defineProps(["remarkContent","remarkTitle"])
// console.log(props,'remarkContent')
</script>
