# 🎨 web_app_front_whut 前端项目  

## 📚 项目名称  
武汉理工大学 Web 应用开发课程实验项目 —— 前端系统  

## 📖 项目简介  

本项目是基于 **Vue3 + ElementPlus** 框架开发的前端 Web 应用，配合后端 **SpringBoot RESTful API**，实现用户注册、登录、修改密码、用户信息列表分页展示等功能。

前端页面实现了：
- 响应式布局，自适应电脑端与移动端  
- ElementPlus 组件库整合，统一美观  
- 用户注册、登录、修改密码界面  
- 用户信息表格分页展示功能  
- 背景切换功能（蓝色、黄色主题）

后端项目仓库：[web_app_end_whut](https://github.com/zzzy-code/web_app_end_whut)

---

## 📂 项目结构  

```
web_app_front_whut/
├── public/                   // 静态资源目录
├── src/
│   ├── api/                  // 封装的 API 请求
│   ├── assets/               // 图片与样式资源
│   ├── components/           // 公共组件
│   ├── views/                // 页面视图
│   ├── App.vue               // 根组件
│   ├── main.js               // 项目入口
│   └── router/               // 路由配置
│
├── vite.config.js            // Vite 配置文件
├── package.json              // 项目依赖
└── README.md                 // 项目说明文件
```

---

## 📦 环境依赖  

| 工具/框架         | 版本       |
|:----------------|:------------|
| Node.js          | 18+         |
| Vite             | 5.x         |
| Vue.js           | 3.x         |
| ElementPlus      | 最新版      |
| Axios            | 最新版      |

---

## 🛠️ 启动与部署  

### 📌 安装依赖  

```bash
npm install
```

### 📌 配置接口地址  

在 `src/api/` 中统一配置后端接口地址，例如：
```javascript
const BASE_URL = "http://localhost:8080/api";
```

### 📌 启动项目  

```bash
npm run dev
```

默认地址：`http://localhost:5173`

---

## 📖 项目功能  

| 功能模块         | 功能说明                                |
|:----------------|:----------------------------------------|
| 用户注册         | 表单验证：用户名、邮箱、生日、密码 |
| 用户登录         | 表单验证 + 调用后端登录接口 |
| 密码找回         | 新密码、确认密码校验 |
| 用户信息分页列表 | el-table+el-pagination 实现分页 |
| 背景切换         | 菜单切换蓝色/黄色主题 |
| 响应式布局       | 适配 PC 与移动端屏幕 |

---
