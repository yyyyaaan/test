<template>
  <div class="home">
    <h1>表格示例</h1>
    <BaseTable :columns="columns" :data="tableData" />
  </div>
</template>

<script setup lang="ts">
import BaseTable from './BaseTable.vue'

const columns = [
  { key: 'name', title: '姓名' },
  { key: 'age', title: '年龄' },
  { key: 'address', title: '地址' }
]

const tableData = [
  { name: '张三', age: 25, address: '北京市' },
  { name: '李四', age: 30, address: '上海市' },
  { name: '王五', age: 28, address: '广州市' }
]
</script> 