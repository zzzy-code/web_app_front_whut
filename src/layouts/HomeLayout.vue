<script setup>
import SidebarNav from "@/components/SidebarNav.vue";
import { onMounted, ref } from "vue";
import { MenuOutlined, DownOutlined } from "@ant-design/icons-vue";
import { ElMessage } from "element-plus";
import { useRouter } from "vue-router";

const theme = ref("blue-theme");
const isMobile = ref(false);
const isOpen = ref(false);
const router = useRouter();
const user = ref({});

function toggleBlueTheme() {
  theme.value = "blue-theme";
  isOpen.value = false;
}
function toggleYellowTheme() {
  theme.value = "yellow-theme";
  isOpen.value = false;
}

function handleLogout() {
  sessionStorage.removeItem("user");
  ElMessage.success("退出登录成功");
  router.push("/auth/login");
}

function handleCalendar() {
  ElMessage.success("查看日历");
  router.push("/home/calendar");
}

function closeDrawer(path) {
  isOpen.value = false;
  router.push(path);
}

function handleResize() {
  isMobile.value = window.innerWidth < 800;
}

onMounted(() => {
  handleResize();
  window.addEventListener("resize", handleResize);
});

onMounted(() => {
  user.value = JSON.parse(sessionStorage.getItem("user"));
});
</script>

<template>
  <el-container class="home-container">
    <el-aside :width="isMobile ? '0px' : '200px'" class="home-aside">
      <el-drawer
          v-model="isOpen"
          direction="ltr"
          size="200px"
          :with-header="false"
          body-class="home-drawer"
          v-if="isMobile"
      >
        <SidebarNav
            :class="theme"
            @toggle-blue-theme="toggleBlueTheme"
            @toggle-yellow-theme="toggleYellowTheme"
            @close-drawer="closeDrawer"
        />
      </el-drawer>
      <SidebarNav
          :class="theme"
          @toggle-blue-theme="toggleBlueTheme"
          @toggle-yellow-theme="toggleYellowTheme"
          @close-drawer="closeDrawer"
          v-else
      />
    </el-aside>
    <el-container class="home-sub-container" :class="theme">
      <el-header class="home-header">
        <el-button v-if="isMobile" link @click="isOpen = !isOpen" color="black">
          <MenuOutlined class="icon" />
        </el-button>
        <el-dropdown class="dropdown">
          <span>
            {{ user.name }}
            <DownOutlined class="down-icon" />
          </span>
          <template #dropdown>
            <el-dropdown-menu>
              <el-dropdown-item @click="handleCalendar">
                日历
              </el-dropdown-item>
              <el-dropdown-item @click="handleLogout">
                退出登录
              </el-dropdown-item>
            </el-dropdown-menu>
          </template>
        </el-dropdown>
      </el-header>
      <el-main class="home-main">
        <router-view v-slot="{ Component }">
          <!-- 使用 Element UI 的淡入淡出过渡效果 -->
          <transition name="el-fade-in-linear" mode="out-in">
            <component :is="Component" />
          </transition>
        </router-view>
      </el-main>
    </el-container>
  </el-container>
</template>

<style scoped>
.home-container {
  width: 100vw;
  height: 100vh;
}

.home-aside {
  user-select: none;
}

.home-sub-container {
  width: 100%;
  height: 100%;
}

.home-header {
  display: flex;
  align-items: center;
  padding: 0 20px;
}

:deep(.home-drawer) {
  padding: 0;
}

.icon {
  font-size: 20px;
}

.el-button {
  outline: none;
}

.dropdown {
  margin-left: auto;
  color: white;
  cursor: pointer;
}

:deep(.el-tooltip__trigger) {
  outline: none;
}

.blue-theme .home-header {
  background-color: var(--el-color-primary-dark-2);
}

.blue-theme .home-main {
  background-color: var(--el-color-primary-light-8);
}

.yellow-theme .home-header {
  background-color: var(--el-color-warning-dark-2);
}

.yellow-theme .home-main {
  background-color: var(--el-color-warning-light-8);
}

/* Element UI 下拉菜单图标旋转效果 */
:deep(.el-dropdown .el-icon) {
  transition: transform 0.3s;
}
:deep(.el-dropdown.is-active .el-icon) {
  transform: rotate(180deg);
}
</style>