<template>
  <div class="table-view">
    <h2>用户信息表格（多级分组）</h2>
    <div class="table-container">
      <BaseTable 
        :columns="columns" 
        :data="tableData" 
        :groupBy="['gender', 'department']"
      />
    </div>
  </div>
</template>

<script setup lang="ts">
import BaseTable from '../components/BaseTable.vue'

const columns = [
  { key: 'gender', title: '性别' },
  { key: 'department', title: '部门' },
  { key: 'age', title: '年龄' },
  { key: 'id', title: 'ID' },
  { key: 'name', title: '姓名' },
  { key: 'position', title: '职位' },
  { key: 'email', title: '邮箱' },
  { key: 'phone', title: '电话' },
  { key: 'address', title: '地址' }
]

const tableData = [
  {
    id: 1,
    name: '张三',
    age: 28,
    gender: '男',
    email: 'zhangsan@example.com',
    phone: '13800138001',
    address: '北京市朝阳区',
    department: '技术部',
    position: '前端工程师'
  },
  {
    id: 2,
    name: '李四',
    age: 32,
    gender: '女',
    email: 'lisi@example.com',
    phone: '13800138002',
    address: '上海市浦东新区',
    department: '产品部',
    position: '产品经理'
  },
  {
    id: 3,
    name: '王五',
    age: 25,
    gender: '男',
    email: 'wangwu@example.com',
    phone: '13800138003',
    address: '广州市天河区',
    department: '设计部',
    position: 'UI设计师'
  },
  {
    id: 4,
    name: '赵六',
    age: 35,
    gender: '女',
    email: 'zhaoliu@example.com',
    phone: '13800138004',
    address: '深圳市南山区',
    department: '运营部',
    position: '运营总监'
  },
  {
    id: 5,
    name: '钱七',
    age: 29,
    gender: '男',
    email: 'qianqi@example.com',
    phone: '13800138005',
    address: '杭州市西湖区',
    department: '技术部',
    position: '后端工程师'
  },
  {
    id: 6,
    name: '孙八',
    age: 31,
    gender: '女',
    email: 'sunba@example.com',
    phone: '13800138006',
    address: '成都市武侯区',
    department: '市场部',
    position: '市场经理'
  },
  {
    id: 7,
    name: '周九',
    age: 27,
    gender: '男',
    email: 'zhoujiu@example.com',
    phone: '13800138007',
    address: '武汉市洪山区',
    department: '销售部',
    position: '销售主管'
  },
  {
    id: 8,
    name: '吴十',
    age: 33,
    gender: '女',
    email: 'wushi@example.com',
    phone: '13800138008',
    address: '南京市玄武区',
    department: '人事部',
    position: 'HR主管'
  }
]
</script>

<style scoped>
.table-view {
  width: 100%;
}

.table-container {
  background: #fff;
  border-radius: 8px;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.1);
  padding: 24px;
}

h2 {
  color: #2c3e50;
  margin-bottom: 24px;
  font-size: 1.5rem;
  font-weight: 600;
}
</style> 