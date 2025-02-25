<template>
  <div class="event-list-page">
    <div class="page-header">
      <h2>事件列表</h2>
      <router-link to="/track/create" class="create-btn">
        创建事件
      </router-link>
    </div>

    <!-- 筛选器 -->
    <div class="filter-section">
      <div class="filter-item">
        <input v-model="searchKey" type="text" placeholder="搜索事件名称或描述" class="search-input">
      </div>
      <div class="filter-item">
        <select v-model="typeFilter" class="filter-select">
          <option value="">全部类型</option>
          <option value="pv">页面浏览</option>
          <option value="click">点击事件</option>
          <option value="custom">自定义事件</option>
        </select>
      </div>
    </div>

    <!-- 事件表格 -->
    <div class="event-table">
      <table>
        <thead>
          <tr>
            <th>事件名称</th>
            <th>事件描述</th>
            <th>事件类型</th>
            <th>属性数量</th>
            <th>今日触发</th>
            <th>创建时间</th>
            <th>操作</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="event in filteredEvents" :key="event.id">
            <td>{{ event.name }}</td>
            <td>{{ event.description }}</td>
            <td>{{ getEventTypeName(event.type) }}</td>
            <td>{{ event.properties.length }}</td>
            <td>{{ formatNumber(event.todayTriggers) }}</td>
            <td>{{ formatDate(event.createdAt) }}</td>
            <td>
              <button class="action-btn edit-btn" @click="startEditEvent(event)">
                编辑
              </button>
              <button class="action-btn delete-btn" @click="deleteEvent(event.id)">
                删除
              </button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- 流失率排行榜 -->
    <div class="churn-rate-ranking">
      <h3>流失率排行榜</h3>
      <table>
        <thead>
          <tr>
            <th>页面</th>
            <th>访问量</th>
            <th>流失量</th>
            <th>流失率</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="page in churnRateRanking" :key="page.page">
            <td>{{ page.page }}</td>
            <td>{{ page.visits }}</td>
            <td>{{ page.exits }}</td>
            <td>{{ page.churnRate }}%</td>
          </tr>
        </tbody>
      </table>
      <!-- 图表展示 -->
      <canvas id="churnRateChart" ref="churnRateChart"></canvas>
    </div>

    <!-- 编辑事件表单 -->
    <div v-if="isEditing" class="edit-event-form">
      <h3>编辑事件</h3>
      <form @submit.prevent="saveEvent">
        <div>
          <label for="eventName">事件名称</label>
          <input v-model="currentEvent.name" id="eventName" type="text" required />
        </div>
        <div>
          <label for="eventDescription">事件描述</label>
          <textarea v-model="currentEvent.description" id="eventDescription" required></textarea>
        </div>
        <div>
          <label for="eventType">事件类型</label>
          <select v-model="currentEvent.type" id="eventType" required>
            <option value="pv">页面浏览</option>
            <option value="click">点击事件</option>
            <option value="custom">自定义事件</option>
          </select>
        </div>
        <button type="submit">保存</button>
        <button type="button" @click="cancelEdit">取消</button>
      </form>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import { formatDate, formatNumber } from '@/utils/format'
import { useRouter } from 'vue-router'
import { Chart, registerables } from 'chart.js'

Chart.register(...registerables)

const router = useRouter()

// 搜索和筛选
const searchKey = ref('')
const typeFilter = ref('')
const isEditing = ref(false)
const currentEvent = ref({})

// 模拟数据
const events = ref([
  {
    id: 1,
    name: 'view_product',
    description: '用户查看商品详情',
    type: 'pv',
    properties: [
      { key: 'product_id', type: 'string', description: '商品ID' },
      { key: 'category', type: 'string', description: '商品分类' },
      { key: 'price', type: 'number', description: '商品价格' }
    ],
    todayTriggers: 12543,
    createdAt: '2024-01-10T08:30:00'
  },
  {
    id: 2,
    name: 'add_to_cart',
    description: '用户添加商品到购物车',
    type: 'click',
    properties: [
      { key: 'product_id', type: 'string', description: '商品ID' },
      { key: 'quantity', type: 'number', description: '添加数量' }
    ],
    todayTriggers: 8976,
    createdAt: '2024-01-12T15:20:00'
  },
  {
    id: 3,
    name: 'start_checkout',
    description: '用户开始结算',
    type: 'custom',
    properties: [
      { key: 'total_amount', type: 'number', description: '总金额' },
      { key: 'item_count', type: 'number', description: '商品数量' }
    ],
    todayTriggers: 5432,
    createdAt: '2024-01-14T11:45:00'
  }
])

// 流失率数据
const pageData = [
  { page: '首页', visits: 1000, exits: 200 },
  { page: '产品详情', visits: 800, exits: 150 },
  { page: '购物车', visits: 600, exits: 300 },
  { page: '结算', visits: 400, exits: 100 },
]

// 计算流失率
const calculateChurnRate = (data) => {
  return data.map(item => {
    const churnRate = ((item.exits / item.visits) * 100).toFixed(2);
    return { ...item, churnRate };
  });
};

// 流失率排行榜
const churnRateRanking = computed(() => {
  const churnRates = calculateChurnRate(pageData);
  churnRates.sort((a, b) => b.churnRate - a.churnRate); // 按流失率降序排序
  return churnRates;
});

// 事件类型名称映射
const getEventTypeName = (type) => {
  const typeMap = {
    pv: '页面浏览',
    click: '点击事件',
    custom: '自定义事件'
  }
  return typeMap[type] || type
}

// 筛选事件列表
const filteredEvents = computed(() => {
  return events.value.filter(event => {
    const matchSearch = searchKey.value ?
      event.name.toLowerCase().includes(searchKey.value.toLowerCase()) ||
      event.description.toLowerCase().includes(searchKey.value.toLowerCase())
      : true
    const matchType = typeFilter.value ? event.type === typeFilter.value : true
    return matchSearch && matchType
  })
})

// 开始编辑事件
const startEditEvent = (event) => {
  currentEvent.value = { ...event } // 复制事件数据
  isEditing.value = true
}

// 保存事件
const saveEvent = () => {
  const index = events.value.findIndex(e => e.id === currentEvent.value.id)
  if (index !== -1) {
    events.value[index] = currentEvent.value // 更新事件数据
  }
  isEditing.value = false // 结束编辑状态
}

// 取消编辑
const cancelEdit = () => {
  isEditing.value = false
}

// 删除事件
const deleteEvent = (id) => {
  if (confirm('确定要删除这个事件吗？')) {
    events.value = events.value.filter(e => e.id !== id)
  }
}

// 绘制图表
const drawChart = () => {
  const ctx = document.getElementById('churnRateChart').getContext('2d');
  const churnRates = churnRateRanking.value;

  const chart = new Chart(ctx, {
    type: 'bar',
    data: {
      labels: churnRates.map(page => page.page),
      datasets: [
        {
          label: '访问量',
          data: churnRates.map(page => page.visits),
          backgroundColor: 'rgba(75, 192, 192, 0.2)',
          borderColor: 'rgba(75, 192, 192, 1)',
          borderWidth: 1
        },
        {
          label: '流失量',
          data: churnRates.map(page => page.exits),
          backgroundColor: 'rgba(255, 99, 132, 0.2)',
          borderColor: 'rgba(255, 99, 132, 1)',
          borderWidth: 1
        },
        {
          label: '流失率',
          data: churnRates.map(page => page.churnRate),
          backgroundColor: 'rgba(255, 206, 86, 0.2)',
          borderColor: 'rgba(255, 206, 86, 1)',
          borderWidth: 1
        }
      ]
    },
    options: {
      scales: {
        y: {
          beginAtZero: true
        }
      }
    }
  });
}

// 在组件挂载后绘制图表
onMounted(() => {
  drawChart();
});
</script>

<style scoped>
.event-list-page {
  padding: 20px;
  margin: 0 auto;
}

.page-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 24px;
}

.create-btn {
  background: #1890ff;
  color: white;
  padding: 8px 16px;
  border-radius: 4px;
  text-decoration: none;
}

.filter-section {
  display: flex;
  gap: 16px;
  margin-bottom: 20px;
}

.filter-item {
  flex: 1;
  max-width: 300px;
}

.search-input,
.filter-select {
  width: 100%;
  padding: 8px 12px;
  border: 1px solid #d9d9d9;
  border-radius: 4px;
}

.event-table {
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  overflow: hidden;
}

table {
  width: 100%;
  border-collapse: collapse;
}

th,
td {
  padding: 12px 16px;
  text-align: left;
  border-bottom: 1px solid #f0f0f0;
}

th {
  background: #fafafa;
  font-weight: 500;
}

.action-btn {
  padding: 4px 8px;
  margin: 0 4px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
  text-decoration: none;
}

.edit-btn {
  background: #faad14;
  color: white;
}

.delete-btn {
  background: #ff4d4f;
  color: white;
}

.action-btn:hover {
  opacity: 0.8;
}

.churn-rate-ranking {
  margin-top: 20px;
}

.churn-rate-ranking table {
  width: 100%;
  border-collapse: collapse;
}

.churn-rate-ranking th,
.churn-rate-ranking td {
  padding: 12px 16px;
  text-align: left;
  border-bottom: 1px solid #f0f0f0;
}

.edit-event-form {
  margin-top: 20px;
  padding: 20px;
  border: 1px solid #d9d9d9;
  border-radius: 4px;
  background: #f9f9f9;
}
</style>