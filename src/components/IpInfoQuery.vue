<template>
  <div class="app-container">
    <div class="container">
      <h1 class="title">IP信息查询</h1>
      <label hidden="hidden">接口地址：</label>
      <input class="input" v-model="serverInput" placeholder="输入后端服务器地址" hidden="hidden"/>

      <div class="dns-container">
        <label>DNS地址：</label>
        <input
            class="input"
            v-model="dnsInput"
            list="dnsList"
            placeholder="输入或选择 DNS 服务器"
        />
        <button class="clearBtn" @click="clearDns">删除</button>
      </div>

      <datalist id="dnsList">
        <option value="8.8.8.8">Google DNS</option>
        <option value="8.8.4.4">Google DNS</option>
        <option value="1.1.1.1">Cloudflare DNS</option>
        <option value="9.9.9.9">Quad9 DNS</option>
        <option value="114.114.114.114">中国 DNS</option>
        <option value="223.5.5.5">阿里云 DNS</option>
        <option value="168.126.63.1">韩国 DNS</option>
      </datalist>

      <div class="ip-container">
        <label>IP地址或域名：</label>
        <input class="input" v-model="ipInput" placeholder="请输入IP地址或域名" />
        <button class="clearBtn" @click="clearIp">删除</button>
      </div>

      <div class="button-container">
        <button class="queryBtn" @click="queryIP">查询信息</button>
        <button class="themeToggle" @click="toggleTheme">切换主题</button>
      </div>

      <div v-if="loading" class="loading">加载中...</div>
      <div v-if="error" class="error">{{ error }}</div>

      <div class="result-container" id="resolvedIPsContainer">
        <div v-for="info in ipInfos" :key="info.ip" class="ip-info-box">
          <p>您查询的信息: <strong>{{ info.ip }}</strong></p>
          <p>国家: <span>{{ info.country || '未知' }}</span></p>
          <p>城市: <span>{{ info.city || '未知' }}</span></p>
        </div>
      </div>
    </div>

    <div class="container">
      <h1 class="title">本地信息</h1>
      <p>IP和位置：<strong>{{ localIP }}</strong></p>
      <p class="clock">{{ clock }}</p>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      ipInput: '',
      dnsInput: '8.8.8.8',
      serverInput: 'http://127.0.0.1:8080/',
      loading: false,
      error: '',
      ipInfos: [],
      localIP: '',
      clock: this.formatTime(new Date()),
      isDarkMode: false, // 追踪深色模式状态
    };
  },
  methods: {
    async queryIP() {
      this.loading = true;
      this.error = '';
      this.ipInfos = [];

      // 清理用户输入，移除 http:// 和 https://，去掉端口号和斜杠
      const sanitizedInput = this.ipInput
          .replace(/^https?:\/\//, '') // 移除 http:// 和 https://
          .replace(/:\d+/, '') // 移除端口号 (例如: :8999)
          .replace(/\/.*$/, ''); // 移除后面的斜杠及其后面的部分

      console.log(`用户输入: ${sanitizedInput}`);
      console.log(`自定义 DNS: ${this.dnsInput}`);
      console.log(`后端服务器地址: ${this.serverInput}`);

      try {
        let ips = [];
        const ipPattern = /^(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)$/;

        if (ipPattern.test(sanitizedInput)) {
          ips.push(sanitizedInput);
        } else {
          const resolvedIPs = await this.resolveDomain(sanitizedInput, this.dnsInput);
          if (resolvedIPs) {
            ips = resolvedIPs;
          } else {
            throw new Error('未能解析域名');
          }
        }

        for (const ip of ips) {
          const response = await fetch(`${this.serverInput}/api/ipinfo/${ip}`);
          if (!response.ok) {
            throw new Error('无法获取 IP 信息');
          }
          const jsonResponse = await response.json();
          this.ipInfos.push(...jsonResponse);
        }
      } catch (err) {
        this.error = err.message;
        console.error('查询 IP 发生错误:', err);
      } finally {
        this.loading = false;
      }
    },
    async resolveDomain(domain, dns_server) {
      console.log(`解析域名: ${domain} 使用 DNS: ${dns_server}`);
      try {
        const response = await fetch(`${this.serverInput}/api/resolve/${domain}?dns=${dns_server}`);
        if (!response.ok) {
          throw new Error('无法解析域名');
        }
        const data = await response.json();
        return data.ip;
      } catch (error) {
        console.error(error);
        return null;
      }
    },
    clearDns() {
      this.dnsInput = '';
    },
    clearIp() {
      this.ipInput = '';
    },
    toggleTheme() {
      this.isDarkMode = !this.isDarkMode; // 切换深色模式状态

      const body = document.body;
      const containers = document.querySelectorAll('.container');
      body.classList.toggle('dark-mode', this.isDarkMode); // 添加或移除深色模式类
      containers.forEach(container => {
        container.classList.toggle('dark-mode', this.isDarkMode);
      });

      body.style.backgroundImage = this.isDarkMode
          ? 'url(https://api.suyanw.cn/api/ys.php)'
          : 'url(https://picsum.photos/1920/1080/?random)';
    },
    formatTime(date) {
      return date.toLocaleTimeString();
    },
  },
  created() {
    setInterval(() => {
      this.clock = this.formatTime(new Date());
    }, 1000);
    this.localIP = '192.168.1.1'; // 占位符，替换为实际方法以获取本地IP
  },
};
</script>

<style scoped>
.app-container {
  font-family: 'Arial', sans-serif;
  padding: 20px;
  background-image: url('https://https://picsum.photos/1920/1080/?random'); /* 设置默认背景 */
  background-repeat: no-repeat;
  background-size: cover;
}

.container {
  margin: 20px auto;
  padding: 20px;
  border: 1px solid #ddd;
  border-radius: 10px;
  background-color: #ffffff;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  max-width: 600px;
}

.title {
  font-size: 28px;
  color: #4CAF50;
  margin-bottom: 15px;
  text-align: center;
}

.input {
  width: calc(100% - 20px);
  padding: 15px;
  border: 1px solid #ccc;
  border-radius: 5px;
  margin-bottom: 15px;
}

.dns-container, .ip-container {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.clearBtn {
  background-color: #f44336;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  padding: 5px 10px;
  transition: background-color 0.3s;
  margin-left: 5px; /* 增加左侧的外边距 */
}

.clearBtn:hover {
  background-color: #e53935;
}

.button-container {
  display: flex;
  justify-content: space-between;
  margin-top: 5px;
}

.queryBtn {
  padding: 10px 15px;
  background-color: #4CAF50;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.3s;
}

.themeToggle {
  padding: 10px 15px;
  background-color: #4CAF50;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.3s;
}

.queryBtn:hover {
  background-color: #45a049;
}

.loading {
  text-align: center;
  margin: 10px 0;
  color: #007BFF;
}

.error {
  color: red;
  text-align: center;
  margin-top: 10px;
}

.result-container {
  margin-top: 15px;
}

.ip-info-box {
  margin: 10px 0;
  padding: 15px;
  border: 1px solid #007BFF;
  border-radius: 5px;
  background-color: rgba(0, 123, 255, 0.1);
  backdrop-filter: blur(10px);
  transition: box-shadow 0.3s;
}

.ip-info-box:hover {
  box-shadow: 0 2px 8px rgba(0, 123, 255, 0.2);
}

.clock {
  font-size: 18px;
  color: #007BFF;
  text-align: center;
  margin-top: 10px;
}

/* 深色模式 */
body.dark-mode {
  background-color: #333;
  color: white;
}

body.dark-mode .container {
  background-color: rgba(0, 0, 0, 0.2); /* 深色背景，带有透明度 */
  border-color: rgba(0, 0, 0, 0.2);
  backdrop-filter: blur(10px); /* 添加毛玻璃效果 */
}

body.dark-mode .input,
body.dark-mode .ip-info-box {
  background-color: rgba(0, 0, 0, 0.2);
  border-color: rgba(0, 0, 0, 0.2);
  color: white;
}

body.dark-mode .queryBtn,
body.dark-mode .clearBtn,
body.dark-mode .themeToggle {
  background-color: #ff4081;
}

body.dark-mode .queryBtn:hover,
body.dark-mode .clearBtn:hover,
body.dark-mode .themeToggle:hover {
  background-color: #f50057;
}

/* 添加深色模式下的浅色文字样式 */
body.dark-mode label {
  color: #ddd; /* 浅色 */
}

body.dark-mode .title {
  color: #80e0a7; /* 浅绿色 */
}
</style>
