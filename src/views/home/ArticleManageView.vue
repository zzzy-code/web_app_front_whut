<script setup>
import { PlusOutlined, SearchOutlined } from "@ant-design/icons-vue";
import { ref, onMounted,watch } from "vue";
import { useRoute, useRouter } from "vue-router";
import { ElMessageBox, ElMessage } from "element-plus";
import axios from "axios";
import ArticleDialog from "@/components/ArticleDialog.vue";

const dialogRef = ref(null);
const route = useRoute();
const router = useRouter();

const user = ref({});
const articles = ref([]);
const currentPage = ref(1);
const pageSize = ref(5);
const total = ref(0);
const keyword = ref("");
const authorName = ref(route.query.name || "");

// 获取用户信息
async function fetchUserInfo() {
  try {
    const response = await axios.get("/api/user/getByName", {
      params: { name: authorName.value }
    });
    if (response.data.code === 200) {
      user.value = response.data.data;
    } else {
      ElMessage.error("获取用户信息失败");
    }
  } catch (error) {
    ElMessage.error("获取用户信息失败");
  }
}

// 获取文章列表
async function fetchArticles() {
  try {
    const response = await axios.get("/api/article/list", {
      params: {
        page: currentPage.value,
        pageSize: pageSize.value,
        keyword: keyword.value,
        authorName: authorName.value
      }
    });
    if (response.data.code === 200) {
      articles.value = response.data.data.article;
      total.value = response.data.data.total;
    } else {
      ElMessage.error("获取文章失败");
    }
  } catch (error) {
    ElMessage.error("获取文章失败");
  }
}

// 搜索功能
function handleSearch() {
  currentPage.value = 1;
  fetchArticles();
}

function handleOpen(title, type, data) {
  dialogRef.value.open(title, type, data);
}

// 新增文章
async function handleAdd(article) {
  try {
    const response = await axios.post("/api/article/add", {
      title: article.title,
      content: article.content,
      name: authorName.value
    });
    if (response.data.code === 200) {
      ElMessage.success("新增成功");
      await fetchArticles();
    } else {
      ElMessage.error("新增失败");
    }
  } catch (error) {
    ElMessage.error("新增失败");
  }
}

// 编辑文章
async function handleEdit(article) {
  try {
    const response = await axios.post("/api/article/edit", {
      id: article.id,
      title: article.title,
      content: article.content
    });
    if (response.data.code === 200) {
      ElMessage.success("修改成功");
      await fetchArticles();
    } else {
      ElMessage.error("修改失败");
    }
  } catch (error) {
    ElMessage.error("修改失败");
  }
}


// 删除文章
function handleDelete(article) {
  ElMessageBox.confirm("确定删除这篇文章吗？", "提示", {
    confirmButtonText: "确定",
    cancelButtonText: "取消",
    type: "warning"
  }).then(async () => {
    try {
      const response = await axios.post('/api/article/delete', { id: article.id })
      if (response.data.code === 200) {
        ElMessage.success("删除成功");
        await fetchArticles();
      }
    } catch (error) {
      ElMessage.error("删除失败");
    }
  });
}

// 页面加载
onMounted(() => {
  if (!authorName.value) {
    ElMessage.error("缺少作者参数");
    router.push("/home/article");
  } else {
    fetchUserInfo();
    fetchArticles();
  }
});

// 监听分页变化
watch([currentPage, pageSize], () => {
  fetchArticles();
}, { immediate: false });

</script>

<template>
  <el-container class="article-manage-container">
    <!-- 上方：用户信息区域 -->
    <el-header class="user-info-header">
      <el-container>
        <el-aside class="front-aside">
          <div class="avatar">
            <el-avatar
                shape="square"
                fit="contain"
                :src="user.avatar"
            />
          </div>
        </el-aside>

        <el-main class="front-main">
          <div class="table">
            <div class="el-descriptions-wrapper">
              <h2 class="el-descriptions-title">作者基本信息</h2>
              <el-descriptions :column="1" border class="el-descriptions-box">
                <el-descriptions-item label="用户名">{{ user.name || '-' }}</el-descriptions-item>
                <el-descriptions-item label="邮箱">{{ user.email || '-' }}</el-descriptions-item>
                <el-descriptions-item label="余额">{{ 0 }}</el-descriptions-item>
                <el-descriptions-item label="生日">{{ user.birthday || '-' }}</el-descriptions-item>
              </el-descriptions>
              <el-button
                  type="primary"
                  plain
                  @click="$router.push('/home/article')"
                  class="back-button"
              >
                返回文章页
              </el-button>
            </div>
          </div>
        </el-main>
      </el-container>
    </el-header>

    <!-- 下方：文章管理区域 -->
    <el-main class="article-manage-main">
      <ArticleDialog ref="dialogRef" @add="handleAdd" @edit="handleEdit" />
      <div class="table-header">
        <el-button
            plain
            type="primary"
            style="height: 40px"
            @click="handleOpen('新增文章', 'add')"
        >
          <PlusOutlined class="icon" />
          新增
        </el-button>
        <el-input
            v-model="keyword"
            placeholder="搜索文章标题"
            :prefix-icon="SearchOutlined"
            @keyup.enter="handleSearch"
            style="width: 250px; margin-left: auto; margin-right: 4px"
        />
        <el-button type="primary" style="height: 40px" @click="handleSearch">
          搜索
        </el-button>
      </div>

      <el-table
          :data="articles"
          style="width: 100%"
          border
          stripe
          :row-style="{ height: '60px' }"
          :show-overflow-tooltip="true"
          :header-row-style="{ height: '48px' }"
          height="100%"
      >
        <el-table-column prop="id" label="序号" width="80" align="center">
          <template #default="scope">
            {{ (currentPage - 1) * pageSize + scope.$index + 1 }}
          </template>
        </el-table-column>
        <el-table-column prop="title" label="文章标题" width="200" align="center" />
        <el-table-column prop="content" label="文章内容" min-width="300" />
        <el-table-column fixed="right" label="操作" width="180" align="center">
          <template #default="scope">
            <el-button
                type="primary"
                size="small"
                @click="handleOpen('修改文章', 'edit', scope.row)"
            >
              编辑
            </el-button>
            <el-button
                type="danger"
                size="small"
                @click="handleDelete(scope.row)"
            >
              删除
            </el-button>
          </template>
        </el-table-column>
      </el-table>

      <el-footer class="article-manage-footer">
        <el-pagination
            v-model:current-page="currentPage"
            v-model:page-size="pageSize"
            :page-sizes="[5, 10, 15, 20]"
            layout="total, sizes, prev, pager, next"
            :total="total"
            background
        />
      </el-footer>
    </el-main>
  </el-container>
</template>

<style scoped>
.article-manage-container {
  height: 100vh;
  display: flex;
  flex-direction: column;
}

.user-info-header {
  display: flex;
  padding: 0;
  margin-bottom: 20px;
  height: auto;
}

.front-aside {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  height: 100%;
}

.front-main {
  flex: 2;
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  height: 100%;
}

.avatar {
  width: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: white;
  border-radius: 4px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  margin-top: 20px;
  margin-bottom: 20px;
}

.el-avatar {
  width: 85%;
  height: 85%;
  border-radius: 4px;
  --el-avatar-bg-color: transition;
}

.table {
  flex: 1;
  width: 100%;
  background-color: white;
  border-radius: 4px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  padding: 12px;
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  overflow: hidden;
}

.el-descriptions-wrapper {
  flex: 1;
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: flex-start;
  margin-top: 36px;
}

.el-descriptions-title {
  font-size: 24px;
  font-weight: bold;
  text-align: center;
  margin-bottom: 24px;
  margin-top: -16px;
  color: #333;
}

.el-descriptions-box {
  min-width: 300px;
  width: 100%;
  height: 100%;
}

.back-button{
  margin-top: 10px;
}

::v-deep(.el-descriptions__label) {
  width: 150px !important;
  text-align: center !important;
  vertical-align: middle !important;
}

::v-deep(.el-descriptions__cell) {
  padding: 12px 10px !important;
  font-size: 16px;
}

.article-manage-main {
  flex: 1;
  padding: 20px;
  background: white;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  display: flex;
  flex-direction: column;
  min-height: 470px;
}

.table-header {
  display: flex;
  width: 100%;
  margin-bottom: 20px;
}

.article-manage-footer {
  display: flex;
  justify-content: center;
  align-items: center;
  margin-top: 20px;
}

.icon {
  margin-right: 4px;
}
</style>