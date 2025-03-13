<template>
    <div class="honeypot-design-container">
        <!-- 左侧导航栏 -->
        <div class="sidebar">
            <div class="logo">
                <!-- <img src="../assets/logo.png" alt="Logo" /> -->
                <span>蜜罐节点配置</span>
            </div>
            <ul class="nav-menu">
                <li class="nav-item active">
                    <i class="icon icon-honeypot"></i>
                    <span>蜜罐节点配置</span>
                </li>
                <li class="nav-item">
                    <i class="icon icon-attacker"></i>
                    <span>攻击智能体</span>
                </li>
                <li class="nav-item">
                    <i class="icon icon-interaction"></i>
                    <span>人机交互演示</span>
                </li>
                <li class="nav-item">
                    <i class="icon icon-algorithm"></i>
                    <span>核心算法配置</span>
                </li>
            </ul>
            <div class="sidebar-footer">
                <i class="icon icon-help"></i>
                <span>帮助</span>
            </div>
        </div>

        <!-- 主内容区 -->
        <div class="main-content">
            <div class="content-header">
                <h2>蜜网设计</h2>
                <div class="editing-info">正在编辑: client1</div>
            </div>

            <!-- 网络拓扑图区域 -->
            <!-- <div class="network-diagram">
                <!-- 这里应该集成一个专门的图形库来渲染网络拓扑 -->
                <!-- <div class="topology-placeholder">
                    <!-- 拓扑图的实际实现会使用如D3.js或vis.js等库 -->
                    <!-- <img src="../assets/network-topology.png" alt="网络拓扑图" /> -->
                    <!-- <div
                        style="height: 300px; background-color: #f5f5f5; display: flex; align-items: center; justify-content: center;">
                        网络拓扑图占位区域
                    </div>
                </div> --> 
                    <!-- 修改网络拓扑图区域 -->
                <div class="network-diagram">
                  <div class="topology-placeholder" ref="networkContainer">
                         <!-- vis-network会在这个容器中渲染 -->
                         </div>

                      <div class="topology-controls">
                      <!-- 其他控制按钮不变 -->
                        </div>
            


                <div class="topology-controls">
                    <button class="btn btn-primary">添加新蜜罐</button>
                    <button class="btn btn-primary">编辑蜜罐</button>
                    <button class="btn btn-danger">移除蜜罐</button>

                    <div class="toggle-container">
                        <span>Show node labels:</span>
                        <label class="toggle">
              <input type="checkbox" v-model="showNodeLabels">
              <span class="toggle-slider"></span>
            </label>
                        <span>{{ showNodeLabels ? 'true' : 'false' }}</span>
                    </div>
                </div>
            </div>

            <!-- 配置面板 -->
            <div class="config-panel">
                <div class="tab-header">
                    <div class="tab" :class="{ active: activeTab === 'basic' }" @click="activeTab = 'basic'">
                        基础配置
                    </div>
                    <div class="tab" :class="{ active: activeTab === 'vulnerability' }"
                        @click="activeTab = 'vulnerability'">
                        漏洞配置
                    </div>
                </div>

                <div class="tab-content">
                    <!-- 基础配置选项卡 -->
                    <div v-if="activeTab === 'basic'">
                        <div class="form-group">
                            <label>蜜罐名称</label>
                            <input type="text" v-model="honeypotName" placeholder="输入蜜罐名称" />
                        </div>

                        <div class="form-group">
                            <label>蜜罐自身价值</label>
                            <input type="number" v-model="honeypotValue" placeholder="输入价值" />
                        </div>

                        <div class="form-group">
                            <label>蜜罐自身量化价值，作为强化奖励的部分参考（整数）</label>
                            <input type="range" v-model="honeypotQuantifiedValue" min="1" max="10" />
                        </div>

                        <div class="form-group">
                            <label>选择远程链接端口</label>
                            <select v-model="selectedPort">
                <option value="">-- 选择端口 --</option>
                <option value="22">SSH (22)</option>
                <option value="80">HTTP (80)</option>
                <option value="443">HTTPS (443)</option>
              </select>
                        </div>
                    </div>

                    <!-- 漏洞配置选项卡 -->
                    <div v-if="activeTab === 'vulnerability'">
                        <div class="form-group">
                            <label>检测到风险等级</label>
                            <input type="text" value="FLAG: Login using insecure SSH user/pass" readonly />
                        </div>

                        <div class="form-group">
                            <label>当蜜罐被探测时输出的日志信息</label>
                            <textarea v-model="logOutput" rows="2"></textarea>
                        </div>

                        <div class="form-group">
                            <label>蜜罐识别能力</label>
                            <input type="text" value="蜜罐在面对攻击者的自身隐藏能力(0.1-1.0)" readonly />
                        </div>
                    </div>

                    <!-- 属性配置区域 -->
                    <div class="attribute-section">
                        <h3>属性配置</h3>
                        <p>节点具有的属性，可能影响进一步的蜜罐性能</p>
                    </div>

                    <!-- 保存按钮 -->
                    <div class="save-section">
                        <button class="btn btn-save" @click="saveSettings">保存设置</button>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import { Network } from "vis-network";
import { DataSet } from "vis-data";

export default {

  name: 'HoneypotDesign',
  data() {
    return {
      activeTab: 'basic',
      honeypotName: 'client1',
      honeypotValue: 5,
      honeypotQuantifiedValue: 7,
      selectedPort: '',
      logOutput: '',
      showNodeLabels: true,
      
      // 添加网络图数据
      network: null,
      nodes: null,
      edges: null
    }
  },
  mounted() {
    // 当组件挂载后初始化网络图
    this.initNetwork();
  },
  methods: {
    saveSettings() {
      // 保存配置的逻辑
      console.log('Settings saved');
    },
    
    initNetwork() {
      // 创建节点数据集
      this.nodes = new DataSet([
        { id: 1, label: 'SLx', x: 0, y: -100, shape: 'box', color: { background: '#c8e6c9' } },
        { id: 2, label: 'Server1', x: -100, y: 0, shape: 'box', color: { background: '#c8e6c9' } },
        { id: 3, label: 'Server2', x: 100, y: 0, shape: 'box', color: { background: '#c8e6c9' } },
        { id: 4, label: 'GitHubProject', x: -50, y: 100, shape: 'box', color: { background: '#c8e6c9' } },
        { id: 5, label: 'OtherProject', x: 50, y: 100, shape: 'box', color: { background: '#c8e6c9' } },
        { id: 6, label: 'client2', x: 150, y: 150, shape: 'box', color: { background: '#c8e6c9' } },
        { id: 7, label: 'client1', x: 0, y: 150, shape: 'box', color: { background: '#ffcc80' } }, // 当前编辑的节点
      ]);
      
      // 创建连线数据集
      this.edges = new DataSet([
        { from: 1, to: 2, arrows: 'to' },
        { from: 1, to: 3, arrows: 'to' },
        { from: 2, to: 4, arrows: 'to' },
        { from: 3, to: 5, arrows: 'to' },
        { from: 4, to: 7, arrows: 'to' },
        { from: 5, to: 6, arrows: 'to' },
        { from: 3, to: 1, arrows: 'to' },
        { from: 4, to: 2, arrows: 'to' },
      ]);
      
      // 配置选项
      const options = {
        physics: {
          enabled: false // 禁用物理引擎以固定节点位置
        },
        nodes: {
          font: {
            size: 14,
            face: 'Arial'
          }
        },
        edges: {
          width: 2,
          color: { color: '#999' }
        },
        interaction: {
          dragNodes: true,
          dragView: true,
          zoomView: true
        }
      };
      
      // 创建网络
      const container = this.$refs.networkContainer;
      const data = { nodes: this.nodes, edges: this.edges };
      this.network = new Network(container, data, options);
      
      // 添加事件监听
      this.network.on('click', (params) => {
        if (params.nodes.length > 0) {
          const nodeId = params.nodes[0];
          console.log('选中节点：', this.nodes.get(nodeId));
          // 这里可以添加选择节点后的处理逻辑
        }
      });
    },
    
    updateNodeLabels() {
      if (this.network) {
        const options = {
          nodes: {
            font: {
              size: this.showNodeLabels ? 14 : 0 // 如果不显示标签，设置字体大小为0
            }
          }
        };
        this.network.setOptions(options);
      }
    }
  },
  watch: {
    showNodeLabels(newVal) {
      this.updateNodeLabels();
    }
  }
}

</script>

<style scoped>
    .honeypot-design-container {
        display: flex;
        height: 100vh;
        overflow: hidden;
    }

    /* 侧边栏样式 */
    .sidebar {
        width: 200px;
        background-color: #f5f5f5;
        border-right: 1px solid #eee;
        display: flex;
        flex-direction: column;
    }

    .logo {
        padding: 20px;
        display: flex;
        align-items: center;
        border-bottom: 1px solid #eee;
    }

    .logo img {
        width: 30px;
        height: 30px;
        margin-right: 10px;
    }

    .nav-menu {
        list-style: none;
        padding: 0;
        margin: 0;
    }

    .nav-item {
        padding: 15px 20px;
        display: flex;
        align-items: center;
        cursor: pointer;
    }

    .nav-item:hover {
        background-color: #e8e8e8;
    }

    .nav-item.active {
        background-color: #e0e0e0;
        border-left: 3px solid #d9534f;
    }

    .icon {
        margin-right: 10px;
        color: #d9534f;
    }

    .sidebar-footer {
        margin-top: auto;
        padding: 15px 20px;
        border-top: 1px solid #eee;
        display: flex;
        align-items: center;
    }

    /* 主内容区域样式 */
    .main-content {
        flex: 1;
        display: flex;
        flex-direction: column;
        overflow: hidden;
    }

    .content-header {
        padding: 20px;
        display: flex;
        justify-content: space-between;
        align-items: center;
        border-bottom: 1px solid #eee;
    }

    .content-header h2 {
        margin: 0;
        font-size: 20px;
        font-weight: normal;
    }

    .editing-info {
        color: #888;
        font-size: 14px;
    }

    /* 网络图区域样式 */
    .network-diagram {
        flex: 1;
        padding: 20px;
        overflow: auto;
        display: flex;
        flex-direction: column;
    }

    .topology-placeholder {
        flex: 1;
        margin-bottom: 20px;
        border: 1px dashed #ccc;
        border-radius: 4px;
        overflow: hidden;
        position: relative; /* 为网络容器添加相对定位 */
        min-height: 300px; /* 确保有足够的高度 */
    }

    .topology-placeholder img {
        width: 100%;
        height: 100%;
        object-fit: contain;
    }

    .topology-controls {
        display: flex;
        gap: 10px;
        align-items: center;
    }

    .btn {
        padding: 8px 15px;
        border: none;
        border-radius: 3px;
        cursor: pointer;
        font-size: 14px;
    }

    .btn-primary {
        background-color: #d9534f;
        color: white;
    }

    .btn-danger {
        background-color: #d9534f;
        color: white;
    }

    .toggle-container {
        margin-left: auto;
        display: flex;
        align-items: center;
        gap: 5px;
    }

    .toggle {
        position: relative;
        display: inline-block;
        width: 50px;
        height: 24px;
    }

    .toggle input {
        opacity: 0;
        width: 0;
        height: 0;
    }

    .toggle-slider {
        position: absolute;
        cursor: pointer;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
        background-color: #ccc;
        transition: .4s;
        border-radius: 24px;
    }

    .toggle-slider:before {
        position: absolute;
        content: "";
        height: 16px;
        width: 16px;
        left: 4px;
        bottom: 4px;
        background-color: white;
        transition: .4s;
        border-radius: 50%;
    }

    input:checked+.toggle-slider {
        background-color: #d9534f;
    }

    input:checked+.toggle-slider:before {
        transform: translateX(26px);
    }

    /* 配置面板样式 */
    .config-panel {
        width: 400px;
        border-left: 1px solid #eee;
        overflow-y: auto;
    }

    .tab-header {
        display: flex;
        background-color: #f7f7f7;
    }

    .tab {
        padding: 15px 20px;
        cursor: pointer;
        text-align: center;
        flex-grow: 1;
        border-bottom: 1px solid #eee;
    }

    .tab.active {
        background-color: white;
        border-bottom: 2px solid #d9534f;
        color: #d9534f;
    }

    .tab-content {
        padding: 20px;
    }

    .form-group {
        margin-bottom: 15px;
    }

    .form-group label {
        display: block;
        margin-bottom: 5px;
        color: #666;
        font-size: 12px;
    }

    .form-group input,
    .form-group select,
    .form-group textarea {
        width: 100%;
        padding: 8px;
        border: 1px solid #ddd;
        border-radius: 3px;
        box-sizing: border-box;
    }

    .attribute-section {
        margin-top: 20px;
        padding-top: 20px;
        border-top: 1px solid #eee;
    }

    .attribute-section h3 {
        font-size: 16px;
        margin-bottom: 10px;
    }

    .attribute-section p {
        color: #888;
        font-size: 12px;
    }

    .save-section {
        margin-top: 20px;
        text-align: center;
    }

    .btn-save {
        background-color: #f0f0f0;
        color: #333;
        padding: 10px 30px;
    }
</style>




