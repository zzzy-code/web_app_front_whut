<script setup>
import * as echarts from 'echarts';
import { useRouter } from 'vue-router';
import { ElMessage } from "element-plus";
import { ref, onMounted, onBeforeUnmount } from 'vue';
import axios from 'axios';

const router = useRouter();
const chart = ref(null);
const userList = ref([]);
const currentPage = ref(1);
const pageSize = ref(5);
const total = ref(0);
const isChartReady = ref(false);

const goToArticleManage = (name) => {
  ElMessage.success(`查看 ${name} 的文章列表`);
  router.push({ path: '/home/articleManage', query: { name } });
};

onMounted(async () => {
  await fetchUserStats();
  initChart();
});

const fetchUserStats = async () => {
  try {
    const response = await axios.get('/api/user/list', {
      params: {
        page: currentPage.value,
        pageSize: pageSize.value
      }
    });

    if (!response.data || typeof response.data !== 'object') {
      throw new Error("API返回的数据不是有效对象");
    }

    const dataList = Array.isArray(response.data.list)
        ? response.data.list
        : [];

    userList.value = dataList.map(item => ({
      id: item?.id ?? '-',
      name: item?.name || '未知用户',
      articleCount: item?.articleCount ?? 0
    }));

    total.value = Number(response.data.total) || 0;
    updateChart();

  } catch (error) {
    console.error('数据加载失败:', error);
    ElMessage.error(`数据加载失败: ${error.message}`);
    userList.value = [];
    total.value = 0;
    updateChart();
  }
};

const updateChart = () => {
  if (!chart.value || !isChartReady.value) return;

  if (!userList.value || userList.value.length === 0) {
    chart.value.setOption({
      title: {
        text: '暂无数据',
        subtext: '请检查后端服务或数据库',
        left: 'center',
        top: 'center'
      },
      xAxis: { show: false },
      yAxis: { show: false }
    }, true);
    return;
  }

  const names = userList.value.map(item => item.name);
  const counts = userList.value.map(item => item.articleCount ?? 0);
  const maxCount = Math.max(...counts, 1);

  const option = {
    tooltip: {
      trigger: 'axis',
      axisPointer: { type: 'shadow' }
    },
    xAxis: {
      type: 'category',
      data: names,
      axisLabel: {
        interval: 0
      }
    },
    yAxis: {
      type: 'value',
      min: 0,
      max: maxCount + 1,
      interval: Math.ceil(maxCount / 5),
      axisLabel: {
        formatter: function (value) {
          return value === 0 ? '0篇' : `${value}篇`;
        }
      },
      name: '文章数量',
      nameLocation: 'end',
      nameTextStyle: {
        fontSize: 14,
        fontWeight: 'bold',
        color: '#333',
        padding: [0, 0, 10, 0]
      }
    },
    series: [{
      name: '文章数量',
      type: 'bar',
      data: counts,
      itemStyle: { color: '#409EFF' },
      barWidth: '40%'
    }]
  };

  chart.value.setOption(option, true);
};

const initChart = () => {
  chart.value = echarts.init(document.querySelector('.chart-container'));
  isChartReady.value = true;
  updateChart();
  window.addEventListener('resize', handleResize);
};

const handleSizeChange = async (size) => {
  pageSize.value = size;
  currentPage.value = 1;
  await fetchUserStats();
};

const handlePageChange = async (page) => {
  currentPage.value = page;
  await fetchUserStats();
};

const handleResize = () => {
  chart.value && chart.value.resize();
};

onBeforeUnmount(() => {
  window.removeEventListener('resize', handleResize);
  chart.value && chart.value.dispose();
});
</script>


<template>
  <el-container class="article-view-container">
    <!-- 左侧边栏 - 用户文章列表 -->
    <el-aside class="article-aside">
      <div class="author-card">
        <h2 class="section-title">用户文章统计</h2>
        <el-table :data="userList" stripe class="author-table" border>
          <el-table-column type="index" label="序号" width="80" align="center" />
          <el-table-column prop="name" label="用户名" width="150" align="center" />
          <el-table-column prop="articleCount" label="文章数量" width="120" align="center" />
          <el-table-column label="操作" align="center">
            <template #default="scope">
              <el-button
                  type="primary"
                  size="small"
                  @click="goToArticleManage(scope.row.name)"
              >
                文章管理
              </el-button>
            </template>
          </el-table-column>
        </el-table>

        <!-- 分页控件 -->
        <div class="pagination-container">
          <el-pagination
              v-model:current-page="currentPage"
              v-model:page-size="pageSize"
              :page-sizes="[5, 10, 15, 20]"
              layout="total, sizes, prev, pager, next"
              :total="total"
              @size-change="handleSizeChange"
              @current-change="handlePageChange"
              background
          />
        </div>
      </div>
    </el-aside>

    <!-- 右侧主内容区 - 图表 -->
    <el-main class="article-main">
      <div class="chart-card">
        <h2 class="section-title">作者发布文章统计图表</h2>
        <h3 class="chart-title">用户发布文章统计</h3>
        <div class="chart-container" style="height: 500px;"></div>
      </div>
    </el-main>
  </el-container>
</template>

<style scoped>
.article-view-container {
  height: 86vh;
  display: flex;
  flex-direction: row;
}

.article-aside {
  width: 50%;
  padding: 12px;
  display: flex;
  flex-direction: column;
  min-width: 400px;
}

.article-main {
  width: 50%;
  padding: 12px;
  display: flex;
  flex-direction: column;
  min-width: 400px;
}

.author-card,
.chart-card {
  width: 100%;
  height: 100%;
  background-color: white;
  border-radius: 4px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  padding: 20px;
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.section-title {
  font-size: clamp(20px, 2vw, 28px);
  font-weight: bold;
  text-align: center;
  margin-bottom: 20px;
  color: #333;
}

.chart-title {
  font-size: clamp(16px, 1.5vw, 20px);
  text-align: left;
  margin-bottom: 20px;
  color: #181818;
}

.author-table {
  margin-top: 15px;
  width: 100%;
  flex-grow: 1;
  min-height: 300px;
}

/* 移除表格行数限制 */
::v-deep(.el-table__body-wrapper) {
  overflow-y: auto;
}

/* 表格行样式 */
::v-deep(.el-table__row) {
  height: 60px;
}

/* 表格单元格样式 */
::v-deep(.el-table__cell) {
  padding: 10px 0;
  font-size: clamp(12px, 1.2vw, 14px);
}

/* 按钮样式 */
::v-deep(.el-button) {
  font-size: clamp(12px, 1.2vw, 14px);
  padding: 8px 12px;
}

.pagination-container {
  margin-top: 15px;
  margin-bottom: 0;
  padding: 10px 0;
  text-align: center;
  display: flex;
  justify-content: center;
  width: 100%;
}

/* 分页器响应式 */
::v-deep(.el-pagination) {
  font-size: clamp(12px, 1.2vw, 14px);
}

.chart-container {
  width: 100%;
  flex-grow: 1;
  min-height: 300px;
}

/* 中等屏幕调整 (992px-1200px) */
@media (max-width: 1200px) {
  .article-aside,
  .article-main {
    padding: 10px;
  }

  .author-card,
  .chart-card {
    padding: 15px;
  }
}

/* 平板设备调整 (768px-992px) */
@media (max-width: 992px) {
  .article-view-container {
    flex-direction: column;
    height: auto;
    min-height: 100vh;
  }

  .article-aside,
  .article-main {
    width: 100%;
    min-width: unset;
    padding: 15px;
    height: auto;
  }

  .author-table {
    min-height: 250px;
  }

  ::v-deep(.el-table__row) {
    height: 50px;
  }
}

/* 手机设备调整 (小于768px) */
@media (max-width: 768px) {
  .section-title {
    font-size: 18px;
    margin-bottom: 15px;
  }

  .chart-title {
    font-size: 16px;
    margin-bottom: 15px;
  }

  .author-card,
  .chart-card {
    padding: 12px;
  }

  ::v-deep(.el-table__cell) {
    padding: 8px 0;
  }

  ::v-deep(.el-button) {
    padding: 6px 10px;
  }

  .pagination-container {
    margin-top: 10px;
  }
}

/* 超小屏幕调整 (小于480px) */
@media (max-width: 480px) {
  .article-aside,
  .article-main {
    padding: 10px;
  }

  .section-title {
    font-size: 16px;
  }

  .chart-title {
    font-size: 14px;
  }

  ::v-deep(.el-table__row) {
    height: 45px;
  }
}
</style>