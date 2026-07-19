<template>
  <div class="app-container">
    <!-- 顶部生成成功浮动 Toast -->
    <transition name="fade">
      <div v-if="showSuccessToast" class="top-success-toast">
        <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round">
          <polyline points="20 6 9 17 4 12"></polyline>
        </svg>
        <span>汽车测评与选车对比方案生成成功！</span>
      </div>
    </transition>

    <!-- 右上角常驻分享按钮 -->
    <button class="floating-share-btn" @click="showShareGuide = true">
      <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="share-icon">
        <circle cx="18" cy="5" r="3"></circle>
        <circle cx="6" cy="12" r="3"></circle>
        <circle cx="18" cy="19" r="3"></circle>
        <line x1="8.59" y1="13.51" x2="15.42" y2="17.49"></line>
        <line x1="15.41" y1="6.51" x2="8.59" y2="10.49"></line>
      </svg>
      <span>分享</span>
    </button>

    <!-- 顶部 App Header (已安全移除未登录提示栏) -->
    <header>
      <h1>{{ appTitle }}</h1>
      <p>智能 AI 车评引擎 · 开启硬核选车对比与落地避坑指南</p>
    </header>

    <!-- 活跃动态 -->
    <UserTicker />

    <!-- 核心卡片 / 结果与历史记录展示 -->
    <main class="glass-card input-group">
      <div class="card-tabs">
        <button class="tab-btn" :class="{ active: !showHistory }" @click="showHistory = false">
          车评设计
        </button>
        <button class="tab-btn" :class="{ active: showHistory }" @click="showHistory = true">
          历史记录 ({{ historyList.length }})
        </button>
      </div>

      <!-- 本地历史记录视图 -->
      <div v-if="showHistory" class="history-view">
        <div class="history-header">
          <span>本地车评历史</span>
          <button v-if="historyList.length > 0" class="clear-all-btn" @click="clearAllHistory">清空全部</button>
        </div>

        <div v-if="historyList.length === 0" class="empty-state">
          <p>暂无汽车测评与选车对比历史记录</p>
        </div>

        <div v-else class="history-grid">
          <div v-for="item in historyList" :key="item.id" class="history-card">
            <div class="h-card-header">
              <span class="h-card-style">{{ item.styleLabel }}</span>
              <span class="h-card-time">{{ item.timestamp }}</span>
            </div>
            
            <div class="h-card-body">
              <div class="h-card-nomad-title">
                <span class="h-city-tag">车型: {{ item.destination }}</span>
                <span class="h-score-badge">车评评分: {{ getAverageScore(item) }}</span>
              </div>

              <p class="h-card-excerpt"><strong>测评摘要：</strong>{{ cleanExcerpt(item.output) }}</p>
            </div>

            <div class="h-card-actions">
              <button class="h-action-btn load-btn" @click="selectHistoryItem(item)">
                加载详情
              </button>
              <button class="h-action-btn delete-btn" @click="deleteHistoryRecord(item.id)">
                删除
              </button>
            </div>
          </div>
        </div>
      </div>

      <div v-else>
        <!-- 主输入区域 (无多余滑动条，保持极简流畅) -->
        <div v-if="!result" class="divination-setup">
          <div class="selector-group">
            <label class="selector-label">目标车型与购车预算范围 (如：预算20-25万对比Model 3与极氪001；15万家用SUV对比宋Plus DM-i与吉利银河L7；30万豪华对比宝马3系与问界M7)</label>
            <input 
              type="text" 
              v-model="destination" 
              class="city-text-input" 
              placeholder="例如：预算20-25万，对比特斯拉 Model 3 与极氪 001"
            />
          </div>

          <div class="selector-group">
            <label class="selector-label">用车场景与配置偏好细节 (如：主要用于城市通勤与带娃郊游、注重续航真实度、智驾辅助、后排空间与保值率)</label>
            <textarea 
              v-model="userInput" 
              placeholder="请输入用车场景与配置偏好。例如：主要用于城市上下班通勤与周末带娃郊游，注重续航真实度、智能驾驶辅助、后排空间与后期保值率。"
              rows="5"
            ></textarea>
          </div>

          <div class="selector-group">
            <label class="selector-label">选择车评流派风格</label>
            <select v-model="activeStyle" class="style-select">
              <option 
                v-for="style in styleOptions" 
                :key="style.value" 
                :value="style.value"
              >
                {{ style.label }}
              </option>
            </select>
          </div>

          <button 
            class="action-btn" 
            :disabled="!destination.trim() || !userInput.trim() || loading" 
            @click="handleGenerate"
          >
            {{ loading ? '车评专家与试驾导师生成中...' : '提交目标车型并生成汽车测评与选车对比方案' }}
          </button>

          <!-- 异常提示 -->
          <div v-if="errorMsg" class="error-banner">
            {{ errorMsg }}
          </div>
        </div>

        <!-- 结果展现 -->
        <div v-else class="divination-result">
          <!-- 方向盘引擎打卡交互板块 -->
          <div class="stamp-section">
            <div class="stamp-canvas">
              <svg 
                class="stamp-svg" 
                :class="{ stamping: isStamping }" 
                @click="stampCheckin" 
                viewBox="0 0 160 160"
              >
                <!-- 方向盘引擎印章 -->
                <circle cx="80" cy="80" r="70" fill="rgba(239, 68, 68, 0.08)" stroke="#ef4444" stroke-width="4" stroke-dasharray="6,3" />
                <circle cx="80" cy="80" r="45" fill="none" stroke="#ef4444" stroke-width="6"/>
                <circle cx="80" cy="80" r="16" fill="#ef4444"/>
                <line x1="80" y1="35" x2="80" y2="64" stroke="#ef4444" stroke-width="6"/>
                <line x1="40" y1="95" x2="66" y2="86" stroke="#ef4444" stroke-width="6"/>
                <line x1="120" y1="95" x2="94" y2="86" stroke="#ef4444" stroke-width="6"/>
                <text x="80" y="142" font-size="12" font-weight="bold" fill="#ef4444" text-anchor="middle">方向盘打卡</text>
              </svg>
              <!-- 漂浮在看动效 (严格无 Emoji) -->
              <transition-group name="float-up">
                <span 
                  v-for="item in floatingItems" 
                  :key="item.id" 
                  class="floating-merit"
                  :style="{ transform: `translate(${item.x}px, ${item.y}px)`, color: '#ef4444' }"
                >
                  {{ item.text }}
                </span>
              </transition-group>
            </div>
            <div class="merit-counter-display">
              累计车评选车打卡：<strong style="color: #ef4444;">{{ totalCheckin }}</strong> 次
              <p class="wood-fish-tip">点击上方方向盘印章，听 V8 引擎声浪为选车梦打卡</p>
            </div>
          </div>

          <!-- 单轨汽车指标多维评估看板 (AI共识判定) -->
          <div v-if="aiScores" class="comparison-dashboard">
            <h3 class="dashboard-title">车评质量多维评估 (AI共识判定)</h3>
            <div class="comparison-grid">
              <div v-for="metric in metricsList" :key="metric.key" class="comparison-row">
                <div class="metric-info">
                  <span class="metric-label">{{ metric.label }}</span>
                  <span class="metric-scores-text">
                    指数值: <strong style="color: var(--accent-color)">{{ aiScores[metric.key] }} / 5</strong>
                  </span>
                </div>
                <div class="comparison-bars">
                  <div class="bar-container">
                    <div class="bar-bg">
                      <div class="bar-fill ai-fill" :style="{ width: aiScores[metric.key] * 20 + '%' }"></div>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <div class="result-header">
            <div class="result-title-group">
              <span class="result-title">汽车测评与选车对比指导</span>
              <span class="success-badge">生成成功</span>
            </div>
            <div class="button-actions">
              <button class="icon-btn" @click="copyText">
                {{ copied ? '已复制全文' : '复制结果' }}
              </button>
              <button class="icon-btn highlight" @click="showShareCard = true">
                生成分享卡片
              </button>
              <button class="icon-btn restart-btn" @click="resetReview">
                重新生成
              </button>
            </div>
          </div>

          <!-- 加载中骨架屏 -->
          <div v-if="loading" class="skeleton">
            <div class="skeleton-line" style="width: 80%"></div>
            <div class="skeleton-line" style="width: 95%"></div>
            <div class="skeleton-line" style="width: 60%"></div>
            <div class="skeleton-line" style="width: 75%"></div>
          </div>

          <!-- 渲染回复 -->
          <div class="ai-response-wrapper">
            <div class="output-content scroll-box" style="text-align: left;">{{ displayResultText }}</div>
          </div>
        </div>
      </div>
    </main>

    <!-- 演示案例区组件 (模块三：30 条经典车评案例展示) -->
    <DemoShowcase @use-sample="handleUseSample" />

    <!-- 底部隐私与服务条款链接 -->
    <footer class="footer-links">
      <button class="footer-link-btn" @click="showPrivacy = true">Privacy Policy</button>
      <button class="footer-link-btn" @click="showTerms = true">Terms of Service</button>
      <button class="footer-link-btn" @click="showContact = true">Contact Us</button>
    </footer>

    <!-- 隐私政策弹窗 -->
    <div v-if="showPrivacy" class="modal-overlay" @click.self="showPrivacy = false">
      <div class="modal-content">
        <h3>Privacy Policy</h3>
        <div class="modal-text-content modal-scroll-area">
          <p>我们非常重视您的隐私。您在本应用中输入的目标车型、预算与偏好等提示词仅用于实时大模型分析与车评报告生成，我们不会在服务器端进行永久存储或记录。</p>
          <p>为了记录您的选车历史、免费额度和打卡数据，本应用会在您的浏览器本地（localStorage）保存相关状态。</p>
        </div>
        <button class="modal-btn" @click="showPrivacy = false">关闭</button>
      </div>
    </div>

    <!-- 服务条款弹窗 -->
    <div v-if="showTerms" class="modal-overlay" @click.self="showTerms = false">
      <div class="modal-content">
        <h3>Terms of Service</h3>
        <div class="modal-text-content modal-scroll-area">
          <p>欢迎使用我们的 AI 汽车测评与选车对比指南专家微应用。生成的测评数据与试驾建议仅供购车参考，建议前往线下4S店实地试驾体验。</p>
        </div>
        <button class="modal-btn" @click="showTerms = false">关闭</button>
      </div>
    </div>

    <!-- 联系我们弹窗 (自适应高度 + weixin.png & dingtalk.png 展示) -->
    <div v-if="showContact" class="modal-overlay" @click.self="showContact = false">
      <div class="modal-content contact-modal-content">
        <h3>Contact Us</h3>
        <div class="modal-text-content contact-card-body">
          <p>如果您在使用过程中遇到任何问题，或有合作意向，可以通过以下方式联系我们：</p>
          <div class="contact-qr-container">
            <div class="contact-qr-card">
              <img :src="weixinImg" alt="微信联系" class="contact-qr-img" />
              <span class="contact-qr-label">微信联系</span>
            </div>
            <div class="contact-qr-card">
              <img :src="dingtalkImg" alt="钉钉交流" class="contact-qr-img" />
              <span class="contact-qr-label">钉钉交流</span>
            </div>
          </div>
          <p class="contact-email">反馈邮箱: <span style="color: var(--primary-color);">us@wuxian.xyz</span></p>
        </div>
        <button class="modal-btn" @click="showContact = false">关闭</button>
      </div>
    </div>

    <!-- 裂变拦截弹窗 (模块四：裂变机制) -->
    <FissionModal 
      :visible="showFission" 
      :wechat-id="wechatId"
      @unlocked="handleUnlocked"
    />

    <!-- 分享卡片弹窗 (模块二扩展) -->
    <ShareCardModal
      :visible="showShareCard"
      :content="displayResultText"
      :wechat-id="wechatId"
      @close="showShareCard = false"
    />

    <!-- 微信 H5 分享引导浮层 -->
    <div v-if="showShareGuide" class="share-guide-overlay" @click="handleShareClose">
      <div class="share-guide-arrow">↗</div>
      <div class="share-guide-content">
        <p>点击右上角菜单 <strong>•••</strong></p>
        <p>选择 <strong>「分享到朋友圈」</strong></p>
        <p class="share-guide-sub">分享这款好用的 AI 汽车测评与选车对比微应用</p>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';
import UserTicker from './components/UserTicker.vue';
import FissionModal from './components/FissionModal.vue';
import DemoShowcase from './components/DemoShowcase.vue';
import ShareCardModal from './components/ShareCardModal.vue';
import appConfig from './config.json';
import weixinImg from '../asset/weixin.png';
import dingtalkImg from '../asset/dingtalk.png';

const appTitle = ref(appConfig.title || '网腾无限AI - 汽车测评与选车对比指南专家');
const wechatId = ref(appConfig.wechatId || 'ai_wuxian_xyz');
const promptTopic = ref(appConfig.promptTopic || '');

const destination = ref('预算20-25万，对比特斯拉 Model 3 与极氪 001');
const userInput = ref('');
const loading = ref(false);
const errorMsg = ref('');
const result = ref('');
const copied = ref(false);
const showSuccessToast = ref(false);
const showFission = ref(false);
const showPrivacy = ref(false);
const showTerms = ref(false);
const showContact = ref(false);
const showShareGuide = ref(false);
const showShareCard = ref(false);

const isStamping = ref(false);
const totalCheckin = ref(parseInt(localStorage.getItem('cp_total_engine_stamp') || '0', 10));
const floatingItems = ref<{ id: number; text: string; x: number; y: number }[]>([]);
let floatingId = 0;

// 车评指标评估列表
const metricsList = [
  { key: 'powerPerformance', label: '动力性能度 (Power)' },
  { key: 'spaceComfort', label: '空间舒适度 (Comfort)' },
  { key: 'smartTech', label: '智能科技度 (Technology)' },
  { key: 'costPerformance', label: '性价比与保值度 (Value & Resale)' },
  { key: 'safetyRigor', label: '安全可靠度 (Safety)' }
];

const aiScores = ref<{ powerPerformance: number; spaceComfort: number; smartTech: number; costPerformance: number; safetyRigor: number; } | null>(null);

const styleOptions = [
  { label: '极客硬核数据参数对比流 (动力三电/零百加速/续航达成率/底盘悬架/芯片)', value: '极客硬核数据参数对比流' },
  { label: '家用舒适与性价比实用流 (后排空间/储物实用性/油耗电耗成本/保值率)', value: '家用舒适与性价比实用流' },
  { label: '深度驾控体验与操纵性能流 (转向精准度/悬架支撑性/NVH隔音/驾驶乐趣)', value: '深度驾控体验与操纵性能流' },
  { label: '新能源智能座舱与智驾探秘流 (语音交互/车机流畅度/NOA领航智驾/安全)', value: '新能源智能座舱与智驾探秘流' },
  { label: '犀利毒舌车评与避坑指引流 (揭露宣传噱头/通病缺陷总结/落地砍价策略)', value: '犀利毒舌车评与避坑指引流' }
];

const activeStyle = ref(styleOptions[0].value);

interface HistoryItem {
  id: string;
  timestamp: string;
  destination: string;
  input: string;
  styleLabel: string;
  aiScores: { powerPerformance: number; spaceComfort: number; smartTech: number; costPerformance: number; safetyRigor: number; } | null;
  output: string;
}

const historyList = ref<HistoryItem[]>([]);
const showHistory = ref(false);

const loadHistory = () => {
  try {
    const raw = localStorage.getItem('cp_history_records');
    historyList.value = raw ? JSON.parse(raw) : [];
  } catch (e) {
    historyList.value = [];
  }
};

const saveHistory = () => {
  localStorage.setItem('cp_history_records', JSON.stringify(historyList.value));
};

const addHistoryRecord = () => {
  const matched = styleOptions.find(o => o.value === activeStyle.value);
  const styleLabel = matched ? matched.label : '车评流派';

  const newItem: HistoryItem = {
    id: Date.now().toString(),
    timestamp: new Date().toLocaleString(),
    destination: destination.value,
    input: userInput.value,
    styleLabel,
    aiScores: aiScores.value ? { ...aiScores.value } : null,
    output: result.value
  };
  historyList.value.unshift(newItem);
  saveHistory();
};

const deleteHistoryRecord = (id: string) => {
  historyList.value = historyList.value.filter(item => item.id !== id);
  saveHistory();
};

const clearAllHistory = () => {
  if (confirm('确认清空所有历史车评报告吗？此操作不可恢复。')) {
    historyList.value = [];
    saveHistory();
  }
};

const selectHistoryItem = (item: HistoryItem) => {
  destination.value = item.destination;
  userInput.value = item.input;
  aiScores.value = item.aiScores ? { ...item.aiScores } : null;
  result.value = item.output;
  showHistory.value = false;
};

const getAverageScore = (item: HistoryItem) => {
  if (!item.aiScores) return '4.0';
  const s = item.aiScores;
  const avg = (s.powerPerformance + s.spaceComfort + s.smartTech + s.costPerformance + s.safetyRigor) / 5;
  return avg.toFixed(1);
};

// 纯前端利用 Web Audio API 动态合成 V8 引擎轰鸣与推背声浪 (锯齿波 D2 73Hz -> A2 110Hz -> D3 146Hz)
const stampCheckin = () => {
  isStamping.value = true;
  setTimeout(() => {
    isStamping.value = false;
  }, 100);

  // 增加打卡计数
  totalCheckin.value++;
  localStorage.setItem('cp_total_engine_stamp', totalCheckin.value.toString());

  // 漂浮动效 (严格无 Emoji)
  const id = floatingId++;
  const x = (Math.random() - 0.5) * 60;
  const y = -40 - Math.random() * 20;
  floatingItems.value.push({ id, text: '引擎轰鸣一路顺风', x, y });
  setTimeout(() => {
    floatingItems.value = floatingItems.value.filter(item => item.id !== id);
  }, 800);

  try {
    const audioCtx = new (window.AudioContext || (window as any).webkitAudioContext)();
    
    // 引擎油门滑音轰鸣 (73Hz -> 110Hz -> 146Hz)
    const osc = audioCtx.createOscillator();
    const gain = audioCtx.createGain();
    osc.type = 'sawtooth'; // 引擎轰鸣质感
    
    const startTime = audioCtx.currentTime;
    osc.frequency.setValueAtTime(73.42, startTime);
    osc.frequency.linearRampToValueAtTime(110.00, startTime + 0.05);
    osc.frequency.linearRampToValueAtTime(146.83, startTime + 0.1);

    gain.gain.setValueAtTime(0.4, startTime);
    gain.gain.exponentialRampToValueAtTime(0.01, startTime + 0.12);

    osc.connect(gain);
    gain.connect(audioCtx.destination);
    osc.start(startTime);
    osc.stop(startTime + 0.13);
  } catch (err) {
    // 捕获异常
  }
};

const parseAIScores = (text: string) => {
  const match = text.match(/\[CHEPING_SCORES\](.*?)\[\/CHEPING_SCORES\]/);
  if (match) {
    const scoreStr = match[1].trim();
    const scores: Record<string, number> = {};
    scoreStr.split(',').forEach(item => {
      const [key, val] = item.split(':');
      if (key && val) {
        scores[key.trim()] = Math.min(5, Math.max(1, parseInt(val.trim(), 10) || 3));
      }
    });
    return {
      powerPerformance: scores.powerPerformance || 3,
      spaceComfort: scores.spaceComfort || 3,
      smartTech: scores.smartTech || 3,
      costPerformance: scores.costPerformance || 3,
      safetyRigor: scores.safetyRigor || 3
    };
  }
  return null;
};

const cleanResponseText = (text: string) => {
  return text.replace(/\[CHEPING_SCORES\][\s\S]*?\[\/CHEPING_SCORES\]/gi, '').trim();
};

const displayResultText = computed(() => {
  return cleanResponseText(result.value);
});

const cleanExcerpt = (text: string) => {
  const cleaned = cleanResponseText(text);
  return cleaned.length > 80 ? cleaned.slice(0, 80) + '...' : cleaned;
};

onMounted(() => {
  loadHistory();
});

const handleShareClose = () => {
  showShareGuide.value = false;
  localStorage.setItem('shared_fission', 'true');
};

const isLimitReached = computed(() => {
  const uses = parseInt(localStorage.getItem('free_uses') || '0', 10);
  const shared = localStorage.getItem('shared_fission') === 'true';
  return uses >= 3 && !shared;
});

const apiEndpoint = import.meta.env.DEV
  ? '/api/local/generate'
  : (import.meta.env.VITE_API_ENDPOINT || 'https://api.wuxian.xyz/api/v1/generate');

const triggerSuccessToast = () => {
  showSuccessToast.value = true;
  setTimeout(() => {
    showSuccessToast.value = false;
  }, 3000);
};

const handleGenerate = async () => {
  if (isLimitReached.value) {
    showFission.value = true;
    return;
  }

  loading.value = true;
  errorMsg.value = '';
  result.value = '';
  aiScores.value = null;

  try {
    const fullPrompt = `【目标车型与购车预算范围】：${destination.value}\n【用车场景与配置偏好细节】：${userInput.value}\n【选定车评流派】：${activeStyle.value}\n${promptTopic.value}`;

    const response = await fetch(apiEndpoint, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      credentials: 'include',
      body: JSON.stringify({
        taskType: 'text',
        prompt: fullPrompt,
        style: activeStyle.value
      })
    });

    const data = await response.json();
    if (data.error) {
      errorMsg.value = data.error;
    } else {
      result.value = data.result;
      aiScores.value = parseAIScores(data.result);
      addHistoryRecord();
      triggerSuccessToast();
      
      const currentUses = parseInt(localStorage.getItem('free_uses') || '0', 10);
      localStorage.setItem('free_uses', (currentUses + 1).toString());
    }
  } catch (err: any) {
    errorMsg.value = '请求接口失败，请检查网络或本地代理服务。';
  } finally {
    loading.value = false;
  }
};

const resetReview = () => {
  result.value = '';
  aiScores.value = null;
};

const handleUseSample = (sampleTopic: string, sampleDestination: string) => {
  // 解析样例填入
  destination.value = sampleDestination;
  userInput.value = sampleTopic;
  showHistory.value = false;
  window.scrollTo({ top: 0, behavior: 'smooth' });
};

const handleUnlocked = () => {
  showFission.value = false;
  handleGenerate();
};

const copyText = async () => {
  try {
    await navigator.clipboard.writeText(displayResultText.value);
    copied.value = true;
    setTimeout(() => {
      copied.value = false;
    }, 2000);
  } catch (err) {
    errorMsg.value = '复制失败，请手动选择复制。';
  }
};
</script>
