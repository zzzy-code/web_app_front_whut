<script setup>
import {computed, ref} from "vue";
import { onMounted } from "vue";
import { CalendarOutlined } from '@ant-design/icons-vue'

const date = ref(new Date());
const user = ref({});

onMounted(() => {
  const storedUser = sessionStorage.getItem("user");
  if (storedUser) {
    user.value = JSON.parse(storedUser);
  }
});

const tableData = computed(() => [
  { label: "用户名", value: user.value.name },
  { label: "邮箱", value: user.value.email },
  { label: "生日", value: user.value.birthday }
]);

</script>

<template>
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
          <h2 class="el-descriptions-title">用户基本信息</h2>
          <el-descriptions
              :column="1"
              border
              class="el-descriptions-box"
          >
            <el-descriptions-item label="用户名">{{ user.name }}</el-descriptions-item>
            <el-descriptions-item label="邮箱">{{ user.email }}</el-descriptions-item>
            <el-descriptions-item label="余额">{{ 0 }}</el-descriptions-item>
            <el-descriptions-item label="生日">
              <CalendarOutlined style="margin-right: 6px; vertical-align: middle;" />
              <span style="vertical-align: middle;">{{ user.birthday }}</span>
            </el-descriptions-item>
          </el-descriptions>
        </div>
      </div>
    </el-main>
  </el-container>
</template>


<style scoped>
.front-aside {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  padding: 48px 0;
  margin-right: 30px;
}

.front-main {
  flex: 2;
  display: flex; /* 添加flex布局 */
  flex-direction: column;
  align-items: flex-start;
  padding: 48px 0;
}

.avatar {
  width: 100%;
  background-color: white;
  border-radius: 4px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  display: flex;
  justify-content: center;
  align-items: center;
  flex: 1;
}

.el-avatar {
  width: 90%;
  height: 90%;
  border-radius: 4px;
  --el-avatar-bg-color: transition;
}


.table {
  flex: 1;
  width: 100%;
  background-color: white;
  border-radius: 4px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  padding: 24px;
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
}

.el-descriptions-wrapper {
  flex: 1;
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center; /* 水平居中 */
  justify-content: flex-start;
  margin-top: 48px; /* 可调节整体偏下的位置 */
}

.el-descriptions-title {
  font-size: 28px;
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

::v-deep(.el-descriptions__label) {
  width: 140px !important;
  text-align: center !important; /* 水平居中 */
  vertical-align: middle !important; /* 垂直居中（默认就是） */
}

::v-deep(.el-descriptions__cell) {
  padding: 14px 10px !important;
  font-size: 16px;
}

</style>
