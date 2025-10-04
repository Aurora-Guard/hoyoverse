<template>
  <div id="app">
    <!-- 导航栏 -->
    <nav class="navbar">
      <div class="nav-brand">我的应用</div>
      <ul class="nav-links">
        <li><a href="#" @click="currentView = 'home'" :class="{ active: currentView === 'home' }">首页</a></li>
        <li v-if="isLoggedIn" class="user-info">
          欢迎，{{ currentUser.username }}！
          <button @click="logout" class="logout-btn">退出</button>
        </li>
      </ul>
    </nav>

    <!-- 主要内容区域 -->
    <main class="main-content">
      <!-- 首页 -->
      <div v-if="currentView === 'home'" class="home-view">
        <h1>欢迎来到hoyo宇宙</h1>
        <div v-if="isLoggedIn" class="welcome-message">
          <p>您已成功登录，可以享受我们的所有功能！</p>
          <div class="features">
            <div class="feature-card" @click="openExternalLink('https://www.bh3.com')">
              <h3>通往崩坏三</h3>
              <div class="link-indicator">外部链接 →</div>
            </div>
            <div class="feature-card" @click="openExternalLink('https://ys.mihoyo.com')">
              <h3>通往原神</h3>
              <div class="link-indicator">外部链接 →</div>
            </div>
            <div class="feature-card" @click="openExternalLink('https://sr.mihoyo.com')">
              <h3>通往崩坏：星穹铁道</h3>
              <div class="link-indicator">外部链接 →</div>
            </div>
            <div class="feature-card" @click="openExternalLink('https://zzz.mihoyo.com')">
              <h3>通往绝区零</h3>
              <div class="link-indicator">外部链接 →</div>
            </div>
            <div class="feature-card" @click="openExternalLink('https://planet.mihoyo.com')">
              <h3>通往星布谷地</h3>
              <div class="link-indicator">外部链接 →</div>
            </div>
            <div class="feature-card" @click="openExternalLink('https://hna.hoyoverse.com')">
              <h3>通往崩坏：因缘精灵</h3>
              <div class="link-indicator">外部链接 →</div>
            </div>
          </div>
        </div>
        <div v-else class="guest-message">
          <p>请登录或注册以使用我们的完整功能</p>
          <div class="action-buttons">
            <button @click="currentView = 'login'" class="btn-primary">登录</button>
            <button @click="currentView = 'register'" class="btn-secondary">注册</button>
          </div>
        </div>
      </div>

      <!-- 登录页面 -->
      <div v-if="currentView === 'login'" class="form-view">
        <h2>用户登录</h2>
        <form @submit.prevent="handleLogin">
          <div class="form-group">
            <label for="login-username">用户名</label>
            <input
              type="text"
              id="login-username"
              v-model="loginForm.username"
              required
              placeholder="请输入用户名"
            >
          </div>
          <div class="form-group">
            <label for="login-password">密码</label>
            <input
              type="password"
              id="login-password"
              v-model="loginForm.password"
              required
              placeholder="请输入密码"
            >
          </div>
          <button type="submit" class="btn-primary">登录</button>
          <p class="form-switch">
            还没有账号？<a href="#" @click="currentView = 'register'">立即注册</a>
          </p>
        </form>
      </div>

      <!-- 注册页面 -->
      <div v-if="currentView === 'register'" class="form-view">
        <h2>用户注册</h2>
        <form @submit.prevent="handleRegister">
          <div class="form-group">
            <label for="register-username">用户名</label>
            <input
              type="text"
              id="register-username"
              v-model="registerForm.username"
              required
              placeholder="请输入用户名"
            >
          </div>
          <div class="form-group">
            <label for="register-email">邮箱</label>
            <input
              type="email"
              id="register-email"
              v-model="registerForm.email"
              required
              placeholder="请输入邮箱"
            >
          </div>
          <div class="form-group">
            <label for="register-password">密码</label>
            <input
              type="password"
              id="register-password"
              v-model="registerForm.password"
              required
              placeholder="请输入密码"
            >
          </div>
          <div class="form-group">
            <label for="register-confirm-password">确认密码</label>
            <input
              type="password"
              id="register-confirm-password"
              v-model="registerForm.confirmPassword"
              required
              placeholder="请再次输入密码"
            >
          </div>
          <button type="submit" class="btn-primary">注册</button>
          <p class="form-switch">
            已有账号？<a href="#" @click="currentView = 'login'">立即登录</a>
          </p>
        </form>
      </div>
    </main>

    <!-- 外部链接确认对话框 -->
    <div v-if="showExternalLinkDialog" class="modal-overlay">
      <div class="modal-dialog">
        <h3>即将离开本站</h3>
        <p>您即将访问外部网站：</p>
        <p class="external-url">{{ pendingExternalLink }}</p>
        <p>请注意，外部网站的内容不在本站控制范围内。</p>
        <div class="dialog-actions">
          <button @click="confirmExternalLink" class="btn-primary">继续访问</button>
          <button @click="cancelExternalLink" class="btn-secondary">取消</button>
        </div>
      </div>
    </div>

    <!-- 消息提示 -->
    <div v-if="message.text" :class="['message', message.type]">
      {{ message.text }}
    </div>
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      currentView: 'home',
      isLoggedIn: false,
      currentUser: {},
      loginForm: {
        username: '',
        password: ''
      },
      registerForm: {
        username: '',
        email: '',
        password: '',
        confirmPassword: ''
      },
      message: {
        text: '',
        type: 'info'
      },
      // 外部链接相关
      showExternalLinkDialog: false,
      pendingExternalLink: '',
      // 模拟用户数据
      users: [
        { id: 1, username: 'demo', email: 'demo@example.com', password: 'demo123' }
      ]
    }
  },
  methods: {
    handleLogin() {
      // 简单的登录验证
      const user = this.users.find(u =>
        u.username === this.loginForm.username &&
        u.password === this.loginForm.password
      );

      if (user) {
        this.isLoggedIn = true;
        this.currentUser = { ...user };
        this.currentView = 'home';
        this.showMessage('登录成功！', 'success');
        this.loginForm.username = '';
        this.loginForm.password = '';
      } else {
        this.showMessage('用户名或密码错误！', 'error');
      }
    },

    handleRegister() {
      // 简单的注册验证
      if (this.registerForm.password !== this.registerForm.confirmPassword) {
        this.showMessage('两次输入的密码不一致！', 'error');
        return;
      }

      if (this.users.some(u => u.username === this.registerForm.username)) {
        this.showMessage('用户名已存在！', 'error');
        return;
      }

      if (this.users.some(u => u.email === this.registerForm.email)) {
        this.showMessage('邮箱已被注册！', 'error');
        return;
      }

      // 创建新用户
      const newUser = {
        id: this.users.length + 1,
        username: this.registerForm.username,
        email: this.registerForm.email,
        password: this.registerForm.password
      };

      this.users.push(newUser);
      this.showMessage('注册成功！请登录', 'success');

      // 清空表单
      this.registerForm = {
        username: '',
        email: '',
        password: '',
        confirmPassword: ''
      };

      // 跳转到登录页面
      this.currentView = 'login';
    },

    logout() {
      this.isLoggedIn = false;
      this.currentUser = {};
      this.currentView = 'home';
      this.showMessage('已成功退出登录', 'info');
    },

    showMessage(text, type = 'info') {
      this.message.text = text;
      this.message.type = type;

      // 3秒后自动清除消息
      setTimeout(() => {
        this.message.text = '';
      }, 3000);
    },

    // 打开外部链接功能
    openExternalLink(url) {
      if (!this.isLoggedIn) {
        this.showMessage('请先登录以使用此功能', 'error');
        this.currentView = 'login';
        return;
      }

      this.pendingExternalLink = url;
      this.showExternalLinkDialog = true;
    },

    confirmExternalLink() {
      window.open(this.pendingExternalLink, '_blank');
      this.showExternalLinkDialog = false;
      this.pendingExternalLink = '';
      this.showMessage('正在打开外部链接...', 'info');
    },

    cancelExternalLink() {
      this.showExternalLinkDialog = false;
      this.pendingExternalLink = '';
    }
  }
}
</script>

<style scoped>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background-color: #f5f7fa;
  color: #333;
  line-height: 1.6;
}

#app {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

/* 导航栏样式 */
.navbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1rem 2rem;
  background-color: #2c3e50;
  color: white;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

.nav-brand {
  font-size: 1.5rem;
  font-weight: bold;
}

.nav-links {
  display: flex;
  list-style: none;
}

.nav-links li {
  margin-left: 1.5rem;
}

.nav-links a {
  color: white;
  text-decoration: none;
  padding: 0.5rem 0.8rem;
  border-radius: 4px;
  transition: background-color 0.3s;
}

.nav-links a:hover,
.nav-links a.active {
  background-color: #34495e;
}

.user-info {
  display: flex;
  align-items: center;
}

.logout-btn {
  background: #e74c3c;
  color: white;
  border: none;
  padding: 0.4rem 0.8rem;
  border-radius: 4px;
  margin-left: 0.8rem;
  cursor: pointer;
  transition: background-color 0.3s;
}

.logout-btn:hover {
  background: #c0392b;
}

/* 主要内容区域 */
.main-content {
  flex: 1;
  padding: 2rem;
  max-width: 1200px;
  margin: 0 auto;
  width: 100%;
}

/* 首页样式 */
.home-view h1 {
  text-align: center;
  margin-bottom: 2rem;
  color: #2c3e50;
}

.welcome-message,
.guest-message {
  text-align: center;
  margin-bottom: 2rem;
}

.features {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 1.5rem;
  margin-top: 2rem;
}

.feature-card {
  background: white;
  padding: 1.5rem;
  border-radius: 8px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  transition: all 0.3s ease;
  cursor: pointer;
  position: relative;
  border: 2px solid transparent;
}

.feature-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 8px 15px rgba(0, 0, 0, 0.15);
  border-color: #3498db;
}

.feature-card h3 {
  color: #3498db;
  margin-bottom: 0.8rem;
}

.link-indicator {
  position: absolute;
  top: 1rem;
  right: 1rem;
  font-size: 0.8rem;
  color: #7f8c8d;
  font-weight: 500;
}

.action-buttons {
  display: flex;
  justify-content: center;
  gap: 1rem;
  margin-top: 1.5rem;
}

/* 表单样式 */
.form-view {
  max-width: 400px;
  margin: 0 auto;
  background: white;
  padding: 2rem;
  border-radius: 8px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.form-view h2 {
  text-align: center;
  margin-bottom: 1.5rem;
  color: #2c3e50;
}

.form-group {
  margin-bottom: 1.2rem;
}

.form-group label {
  display: block;
  margin-bottom: 0.5rem;
  font-weight: 500;
}

.form-group input {
  width: 100%;
  padding: 0.8rem;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 1rem;
  transition: border-color 0.3s;
}

.form-group input:focus {
  border-color: #3498db;
  outline: none;
}

/* 按钮样式 */
.btn-primary,
.btn-secondary {
  padding: 0.8rem 1.5rem;
  border: none;
  border-radius: 4px;
  font-size: 1rem;
  cursor: pointer;
  transition: background-color 0.3s;
  width: 100%;
  margin-top: 0.5rem;
}

.btn-primary {
  background-color: #3498db;
  color: white;
}

.btn-primary:hover {
  background-color: #2980b9;
}

.btn-secondary {
  background-color: #95a5a6;
  color: white;
}

.btn-secondary:hover {
  background-color: #7f8c8d;
}

.form-switch {
  text-align: center;
  margin-top: 1.5rem;
}

.form-switch a {
  color: #3498db;
  text-decoration: none;
}

.form-switch a:hover {
  text-decoration: underline;
}

/* 外部链接确认对话框 */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1001;
}

.modal-dialog {
  background: white;
  padding: 2rem;
  border-radius: 8px;
  max-width: 500px;
  width: 90%;
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.2);
}

.modal-dialog h3 {
  color: #2c3e50;
  margin-bottom: 1rem;
}

.external-url {
  background: #f8f9fa;
  padding: 0.8rem;
  border-radius: 4px;
  font-family: monospace;
  word-break: break-all;
  margin: 1rem 0;
  border-left: 4px solid #3498db;
}

.dialog-actions {
  display: flex;
  gap: 1rem;
  margin-top: 1.5rem;
}

.dialog-actions button {
  width: auto;
  flex: 1;
}

/* 消息提示样式 */
.message {
  position: fixed;
  top: 20px;
  right: 20px;
  padding: 1rem 1.5rem;
  border-radius: 4px;
  color: white;
  font-weight: 500;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  z-index: 1000;
}

.message.info {
  background-color: #3498db;
}

.message.success {
  background-color: #2ecc71;
}

.message.error {
  background-color: #e74c3c;
}

/* 响应式设计 */
@media (max-width: 768px) {
  .navbar {
    flex-direction: column;
    padding: 1rem;
  }

  .nav-links {
    margin-top: 1rem;
    flex-wrap: wrap;
    justify-content: center;
  }

  .nav-links li {
    margin: 0.3rem;
  }

  .main-content {
    padding: 1rem;
  }

  .features {
    grid-template-columns: 1fr;
  }

  .action-buttons {
    flex-direction: column;
    align-items: center;
  }

  .action-buttons button {
    width: 100%;
    max-width: 200px;
  }

  .dialog-actions {
    flex-direction: column;
  }
}
</style>

