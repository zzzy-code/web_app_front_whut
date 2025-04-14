<script setup></script>

<template>
  <el-container class="auth-container">
    <el-main class="auth-main">
      <router-view v-slot="{ Component }">
        <transition name="el-fade-in-linear" mode="out-in">
          <component :is="Component" />
        </transition>
      </router-view>
    </el-main>
  </el-container>
</template>

<style scoped>
.auth-container {
  height: 100vh;
  justify-content: center;
  align-items: center;
  font-size: 12px;
  background-image: url('@/assets/LoginBackground.jpg');
  background-size: cover;
  background-position: center;
  color: black;
}

.auth-main {
  flex: none;
  background-color: rgba(255, 255, 255, 0.9);
  border-radius: 24px;
  padding: 12px 25px;
  overflow: hidden;
  height: auto !important;
}
</style>