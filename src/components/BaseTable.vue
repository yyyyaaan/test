<template>
  <div class="base-table">
    <table>
      <thead>
        <tr>
          <th v-for="column in columns" :key="column.key" :class="column.type">
            {{ column.title }}
          </th>
        </tr>
      </thead>
      <tbody>
        <transition-group name="list">
          <template v-for="(genderGroup, gender) in groupedData" :key="gender">
            <!-- 性别分组行（未展开） -->
            <tr v-if="!expandedGroups[gender]" :key="`${gender}-collapsed`" class="group-row" @click="toggleGroup(gender)">
              <td class="group group-cell">
                <div class="group-content">
                  <span class="expand-icon">{{ expandedGroups[gender] ? '▼' : '▶' }}</span>
                  {{ gender }}
                </div>
              </td>
              <td class="data">平均{{ getAverageAge(getAllItems(genderGroup)) }}岁</td>
            </tr>

            <!-- 性别分组展开后显示部门分组 -->
            <template v-if="expandedGroups[gender]">
              <template v-for="(deptGroup, dept, deptIndex) in genderGroup" :key="`${gender}-${dept}`">
                <!-- 第一个部门 -->
                <tr v-if="deptIndex === 0">
                  <td :rowspan="getGenderRowSpan(genderGroup, gender)" 
                      class="group group-cell clickable"
                      @click="toggleGroup(gender)">
                    <div class="group-content">
                      <span class="expand-icon">{{ expandedGroups[gender] ? '▼' : '▶' }}</span>
                      {{ gender }}
                    </div>
                  </td>
                  <td class="group group-cell clickable" 
                      :rowspan="expandedSubGroups[`${gender}-${dept}`] ? deptGroup.length : 1"
                      @click="toggleSubGroup(`${gender}-${dept}`)">
                    <div class="group-content">
                      <span class="expand-icon">
                        {{ expandedSubGroups[`${gender}-${dept}`] ? '▼' : '▶' }}
                      </span>
                      {{ dept }}
                    </div>
                  </td>
                  <template v-if="!expandedSubGroups[`${gender}-${dept}`]">
                    <td class="data">平均{{ getAverageAge(deptGroup) }}岁</td>
                  </template>
                  <template v-else>
                    <td v-for="column in columns.slice(2)" :key="column.key" :class="column.type">
                      {{ deptGroup[0][column.key] }}
                    </td>
                  </template>
                </tr>

                <!-- 第一个部门的数据行 -->
                <template v-if="expandedSubGroups[`${gender}-${dept}`]">
                  <tr v-for="(item, index) in deptGroup.slice(1)" :key="index" class="data-row">
                    <td v-for="column in columns.slice(2)" :key="column.key" :class="column.type">
                      {{ item[column.key] }}
                    </td>
                  </tr>
                </template>

                <!-- 其他部门 -->
                <tr v-if="deptIndex > 0">
                  <td class="group group-cell clickable" 
                      :rowspan="expandedSubGroups[`${gender}-${dept}`] ? deptGroup.length : 1"
                      @click="toggleSubGroup(`${gender}-${dept}`)">
                    <div class="group-content">
                      <span class="expand-icon">{{ expandedSubGroups[`${gender}-${dept}`] ? '▼' : '▶' }}</span>
                      {{ dept }}
                    </div>
                  </td>
                  <template v-if="!expandedSubGroups[`${gender}-${dept}`]">
                    <td class="data">平均{{ getAverageAge(deptGroup) }}岁</td>
                  </template>
                  <template v-else>
                    <td v-for="column in columns.slice(2)" :key="column.key" :class="column.type">
                      {{ deptGroup[0][column.key] }}
                    </td>
                  </template>
                </tr>

                <!-- 其他部门的数据行 -->
                <template v-if="deptIndex > 0 && expandedSubGroups[`${gender}-${dept}`]">
                  <tr v-for="(item, index) in deptGroup.slice(1)" :key="index" class="data-row">
                    <td v-for="column in columns.slice(2)" :key="column.key" :class="column.type">
                      {{ item[column.key] }}
                    </td>
                  </tr>
                </template>
              </template>
            </template>
          </template>
        </transition-group>
      </tbody>
    </table>
  </div>
</template>

<script setup lang="ts">
import { computed, ref } from 'vue'

type ColumnType = 'group' | 'data' | 'detail'

interface Column {
  key: string;
  title: string;
  type: ColumnType;
}

interface Props {
  columns?: Column[];
  data?: any[];
}

const props = withDefaults(defineProps<Props>(), {
  columns: () => [],
  data: () => []
})

// 展开/收起状态
const expandedGroups = ref<{ [key: string]: boolean }>({})
const expandedSubGroups = ref<{ [key: string]: boolean }>({})
  
// 切换分组展开/收起
const toggleGroup = (groupKey: string) => {
  expandedGroups.value[groupKey] = !expandedGroups.value[groupKey]
}

const toggleSubGroup = (groupKey: string) => {
  expandedSubGroups.value[groupKey] = !expandedSubGroups.value[groupKey]
}

// 获取分组内所有项目
const getAllItems = (group: Record<string, any[]>): any[] => {
  return Object.values(group).flat()
}

// 计算平均年龄
const getAverageAge = (items: any[]) => {
  if (!items || items.length === 0) return 0
  const totalAge = items.reduce((sum, item) => sum + (item.age || 0), 0)
  return Math.round(totalAge / items.length)
}

// 按性别和部门分组数据
const groupedData = computed(() => {
  const groups: Record<string, Record<string, any[]>> = {}
  
  props.data.forEach(item => {
    const gender = item.gender || '未知'
    const dept = item.department || '未知'
    
    if (!groups[gender]) {
      groups[gender] = {}
    }
    if (!groups[gender][dept]) {
      groups[gender][dept] = []
    }
    
    groups[gender][dept].push(item)
  })

  return groups
})

// 计算性别分组的总行数
const getGenderRowSpan = (genderGroup: Record<string, any[]>, gender: string) => {
  let totalRows = Object.keys(genderGroup).length // 每个部门的标题行

  // 遍历所有部门，如果部门展开则加上其数据行数
  for (const [dept, items] of Object.entries(genderGroup)) {
    const groupKey = `${gender}-${dept}`
    if (expandedSubGroups.value[groupKey]) {
      totalRows += items.length - 1 // 减1是因为第一行已经在部门标题行中计算过了
    }
  }

  console.log('Departments:', Object.keys(genderGroup).length)
  console.log('Total rows:', totalRows)
  return totalRows
}

// 获取详细信息列
const detailColumns = computed(() => {
  return props.columns.filter(col => col.type === 'detail')
})
</script>

<style scoped>
.base-table {
  width: 100%;
  overflow-x: auto;
  background: white;
}

table {
  width: 100%;
  border-collapse: collapse;
  table-layout: fixed;
  transition: none;
}

th, td {
  padding: 12px 16px;
  text-align: left;
  border-bottom: 1px solid #ebeef5;
  word-break: break-all;
  white-space: normal;
}

th {
  background-color: #f5f7fa;
  font-weight: 500;
  color: #606266;
}

td {
  color: #606266;
  transition: all 0.3s ease;
}

/* 分组单元格样式 */
.group-cell {
  vertical-align: middle !important;
  text-align: center;
  position: relative;
  border-right: 1px solid #ebeef5;
  transition: none;
}

.group-content {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100%;
  padding: 0 32px 0 16px;
  position: relative;
  transition: all 0.3s ease;
}

.expand-icon {
  position: absolute;
  right: 8px;
  top: 50%;
  transform: translateY(-50%);
  font-size: 12px;
  color: #909399;
  transition: transform 0.3s ease;
}

/* 设置固定的列宽 */
th:nth-child(1), td:nth-child(1) {
  width: 120px; /* 性别列 */
}

th:nth-child(2), td:nth-child(2) {
  width: 120px; /* 部门列 */
}

th:nth-child(3), td:nth-child(3) {
  width: 100px; /* 年龄列 */
}

th:nth-child(n+4), td:nth-child(n+4) {
  width: 160px; /* 其他详细信息列 */
}

/* 确保合并单元格的边框显示正确 */
td[rowspan] {
  border-right: 1px solid #ebeef5;
  transition: none;
}

/* 数据行样式 */
.data-row td:first-child {
  border-left: 1px solid #ebeef5;
  transition: none;
}

/* 可点击元素样式 */
.clickable {
  cursor: pointer;
}

/* 添加过渡效果 */
tr {
  transition: all 0.3s ease;
}

tbody {
  position: relative;
}

/* 修改过渡动画样式 */
.list-enter-active{
  transition: all 0.3s ease;
}
.list-leave-active {
  transition: all 0.3s ease;
}

.list-enter-from {
  opacity: 0.3;
  transform: translateY(20px);
}

.list-leave-to {
  opacity: 0.3;
  transform: translateY(20px);
}

/* 确保移动过渡平滑 */
/* .list-move {
  transition: transform 0.5s ease;
} */

/* 优化展开图标过渡 */
.expand-icon {
  transition: transform 0.3s ease;
}

.expanded .expand-icon {
  transform: translateY(-50%) rotate(180deg);
}

/* 优化行高度变化 */
tr {
  transition: all 0.3s ease;
}

td {
  transition: all 0.3s ease;
}

.group-content {
  transition: all 0.3s ease;
}

/* 确保过渡期间内容不会溢出 */
.base-table {
  overflow: hidden;
}

tbody {
  position: relative;
}

/* 隐藏未展开状态下不需要显示的单元格内容 */
tr[data-collapsed="true"] td:nth-child(n+4) {
  padding: 0;
  border: none;
  height: 0;
  overflow: hidden;
}
</style> 