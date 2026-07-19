<template>
  <section class="glass-card showcase-container">
    <div class="showcase-header">
      <div class="showcase-title-group">
        <h2 class="showcase-title">汽车测评与选车对比案例库 (30 精选样例)</h2>
        <p class="showcase-subtitle">体验极客硬核参数、家用实用性价比、深度驾控底盘、智能座舱智驾与毒舌避坑指南，点击“一键同款生成”即可即刻核算</p>
      </div>
      <div class="showcase-badge">客观避坑 · 免费体验</div>
    </div>

    <!-- 搜索与分类筛选 -->
    <div class="showcase-filter-bar">
      <div class="category-tabs">
        <button 
          v-for="cat in categories" 
          :key="cat"
          class="category-tab"
          :class="{ active: currentCategory === cat }"
          @click="currentCategory = cat"
        >
          {{ cat }}
        </button>
      </div>
      <div class="search-input-wrapper">
        <input 
          v-model="searchQuery"
          type="text"
          placeholder="搜索目标车型、预算范围、续航真实度、底盘智驾、车评流派或关键字..."
          class="search-input"
        />
      </div>
    </div>

    <!-- 样例列表格 Grid -->
    <div class="sample-grid">
      <div 
        v-for="sample in paginatedSamples" 
        :key="sample.id" 
        class="sample-card"
      >
        <div class="sample-card-header">
          <span class="topic-category-tag">{{ sample.category }}</span>
          <span class="style-name-tag">{{ sample.style }}</span>
        </div>
        <div class="sample-original">
          <span class="sample-label">选车诉求：</span>“{{ sample.destination }}，需求：{{ sample.topic }}”
        </div>
        <div class="sample-rewritten">
          <span class="sample-label">测评结论：</span>{{ sample.core }}
        </div>
        <div class="sample-card-footer">
          <button class="use-sample-btn" @click="$emit('use-sample', sample.topic, sample.destination)">
            一键同款生成
          </button>
        </div>
      </div>
    </div>

    <!-- 空状态提示 -->
    <div v-if="filteredSamples.length === 0" class="empty-showcase">
      未找到匹配的车评案例，请尝试切换分类或重置搜索关键词。
    </div>

    <!-- 分页组件 -->
    <div v-if="filteredSamples.length > pageSize" class="pagination-bar">
      <button 
        class="page-btn" 
        :disabled="currentPage === 1"
        @click="currentPage--"
      >
        上一页
      </button>
      <span class="page-info">第 {{ currentPage }} / {{ totalPages }} 页 (共 {{ filteredSamples.length }} 条)</span>
      <button 
        class="page-btn" 
        :disabled="currentPage === totalPages"
        @click="currentPage++"
      >
        下一页
      </button>
    </div>
  </section>
</template>

<script setup lang="ts">
import { ref, computed, watch } from 'vue';

defineEmits<{
  (e: 'use-sample', text: string, destination: string): void;
}>();

const categories = ['全部', '纯电新能源', '插混增程', '燃油豪华', '家用SUV', '代步紧凑'];
const currentCategory = ref('全部');
const searchQuery = ref('');
const currentPage = ref(1);
const pageSize = 6;

interface ChepingSample {
  id: number;
  category: string;
  destination: string;
  topic: string;
  style: string;
  core: string;
}

// 精选 30 条汽车测评与选车对比案例
const raw30Samples: ChepingSample[] = [
  {
    id: 1,
    category: '纯电新能源',
    destination: '预算20-25万对比特斯拉 Model 3 与极氪 001',
    topic: '关注城市通勤与高速长途、续航达成率、操控性与智驾体验。',
    style: '极客硬核数据参数对比流',
    core: '结论：Model 3 能耗极致、车机响应快但后排空间紧凑；极氪 001 底盘质感出众、空间巨大但车机生态稍逊。强烈建议带家人试驾后排。'
  },
  {
    id: 2,
    category: '插混增程',
    destination: '15万家用SUV对比比亚迪宋 Plus DM-i 与吉利银河 L7',
    topic: '关注后备箱空间、油耗表现、底盘悬架与长途舒适度。',
    style: '家用舒适与性价比实用流',
    core: '结论：宋 Plus DM-i 市场保有量大保值率稳定，DM-i技术成熟；银河 L7 动力储备更强，三档DHT高速再加速有优势，看重驾驶动力选银河，看重稳妥选宋。'
  },
  {
    id: 3,
    category: '插混增程',
    destination: '30万大六座奶爸车对比问界 M7 与理想 L7/L8',
    topic: '关注二三排空间、华为智驾鸿蒙座舱、冰箱彩电大沙发体验。',
    style: '新能源智能座舱与智驾探秘流',
    core: '结论：问界 M7 华为ADS 2.0智驾与鸿蒙生态无敌，安全碰撞防护硬核；理想 L7/L8 底盘调校更偏向舒适大沙发，三排空间设计更具人情味。'
  },
  {
    id: 4,
    category: '燃油豪华',
    destination: '30万中型豪华轿车对比宝马 3系、奔驰 C级 与奥迪 A4L',
    topic: '关注品牌面子、操控乐趣、内饰豪华感与保养落地折扣。',
    style: '深度驾控体验与操纵性能流',
    core: '结论：3系兼顾操控与底盘舒适，油门响应线性；C级内饰氛围灯豪华天花板但1.5T动力一般；A4L性价比最高，机械四驱稳定。'
  },
  {
    id: 5,
    category: '代步紧凑',
    destination: '10万纯电代步小车对比比亚迪海豚与五菱缤果 Plus',
    topic: '关注接送孩子、菜市场停车、续航真实度与安全气囊配置。',
    style: '犀利毒舌车评与避坑指引流',
    core: '结论：海豚 e平台3.0底盘素质完胜，刀片电池安全性高；缤果 Plus 空间利用率惊人但底盘滤震偏硬。避坑提示：切记不要选最入门无ESP版本。'
  },
  {
    id: 6,
    category: '纯电新能源',
    destination: '15-18万纯电轿跑对比深蓝 SL03 与小鹏 P5/MONA M03',
    topic: '关注外观无框车门、智能驾驶辅助与年轻高颜值。',
    style: '极客硬核数据参数对比流',
    core: '结论：深蓝 SL03 后驱操纵感强，掀背造型拉风；小鹏 MONA M03 自动泊车与智驾门槛极低。重视颜值选深蓝，看重智驾选小鹏。'
  },
  {
    id: 7,
    category: '家用SUV',
    destination: '预算25-30万燃油/混动SUV对比汉兰达与领克 09',
    topic: '关注7座空间、保值率、安全用料与长途自驾游。',
    style: '家用舒适与性价比实用流',
    core: '结论：汉兰达双擎极其省油且保值率神话，但车机内饰简陋；领克 09 SPA架构安全用料天花板，操控底盘越级表现。'
  },
  {
    id: 8,
    category: '纯电新能源',
    destination: '25-30万纯电SUV对比蔚来 ES6、小鹏 G9 与阿维塔 11',
    topic: '关注换电便利性、800V高压快充与智驾领航。',
    style: '新能源智能座舱与智驾探秘流',
    core: '结论：蔚来 ES6 具备独特换电服务与女主人副驾；小鹏 G9 800V快充与S4超快充极佳；阿维塔 11 华为三激光雷达外观前卫。'
  },
  {
    id: 9,
    category: '燃油豪华',
    destination: '50万中大型豪华SUV对比宝马 X5、奔驰 GLE 与沃尔沃 XC90',
    topic: '关注国产加长后排、品牌气场、环保无异味与安全性。',
    style: '深度驾控体验与操纵性能流',
    core: '结论：国产X5加长轴距后排空间称霸，3.0T直六发动机丝滑；XC90性价比天花板且环保无异味；GLE星辉气场最强。'
  },
  {
    id: 10,
    category: '代步紧凑',
    destination: '8-10万燃油代步车对比吉利帝豪、长安逸动与日产轩逸',
    topic: '关注皮实耐用、省油维修便宜与皮质座椅。',
    style: '家用舒适与性价比实用流',
    core: '结论：轩逸经典省油代步但配置极低车身轻；帝豪与逸动全系配置大满贯，内饰质感与静音远超合资。推荐首选国产自主。'
  },
  {
    id: 11,
    category: '纯电新能源',
    destination: '35万中大型纯电对比蔚来 ET7、小米 SU7 与保时捷 Taycan 避坑对比',
    topic: '关注防晒车顶、0-100加速、赛道性能与智能座舱。',
    style: '极客硬核数据参数对比流',
    core: '结论：小米 SU7 Max 赛道性能与生态互联无敌，防晒效果极佳；蔚来 ET7 行政质感出众；Taycan 品牌底蕴深但选装昂贵。'
  },
  {
    id: 12,
    category: '插混增程',
    destination: '12-15万插混轿车对比秦 L DM-i、风云 A8 与吉利银河 L6',
    topic: '关注满油满电2000公里续航、百公里2.9L油耗真实性。',
    style: '犀利毒舌车评与避坑指引流',
    core: '结论：秦 L 五代DM技术油耗惊人且底盘升级E型多连杆；银河 L6 动力凶猛适合高速；风云 A8 电池用料扎实。建议实测亏电油耗。'
  },
  {
    id: 13,
    category: '家用SUV',
    destination: '20万合资燃油SUV对比本田 CR-V、丰田 RAV4 与大众途观L',
    topic: '关注后排平整度、日系混动保值率与德系双离合稳定性。',
    style: '家用舒适与性价比实用流',
    core: '结论：CR-V 空间利用魔术师，后门可90度开启；途观L 2.0T动力充沛高速稳重；RAV4 混动E-Four四驱通过性好。'
  },
  {
    id: 14,
    category: '燃油豪华',
    destination: '40万豪华C级轿车对比奥迪 A6L、宝马 5系 与奔驰 E级',
    topic: '关注商用行政气场、后排尊享座椅与落地优惠折扣。',
    style: '深度驾控体验与操纵性能流',
    core: '结论：A6L 45TFSI quattro 性价比最高且宜商宜家；E级大连屏奢华气场足；5系改款外观见仁见智，后排31寸大屏吸引眼球。'
  },
  {
    id: 15,
    category: '代步紧凑',
    destination: '5-7万城市代步纯电对比五菱宏光 MINIEV、吉利熊猫 MINI 与长安 Lumin',
    topic: '关注买菜接娃、充电方便度与空调制冷速度。',
    style: '犀利毒舌车评与避坑指引流',
    core: '结论：Lumin 空间最阔绰且配备双气囊；熊猫 MINI 快充版20分钟满电；宏光 MINIEV 零部件极便宜。避坑：务必选择带快充版本。'
  },
  {
    id: 16,
    category: '纯电新能源',
    destination: '20万纯电猎装车对比极氪 007、智界 S7 与星纪元 ES',
    topic: '关注800V架构、前双叉臂后多连杆悬架与CDC空气悬架。',
    style: '极客硬核数据参数对比流',
    core: '结论：007 全系800V且自研金砖电池性价比爆棚；智界 S7 鸿蒙智驾与无人代客泊车亮点足；星纪元 ES 底盘云台质感舒适。'
  },
  {
    id: 17,
    category: '插混增程',
    destination: '20-25万硬派越野对比方程豹 豹5、坦克 300 Hi4-T 与捷途山海 T2',
    topic: '关注非承载式车身、云辇-P底盘、三把锁与露营放电。',
    style: '深度驾控体验与操纵性能流',
    core: '结论：豹5 DMO平台兼顾城市舒适与超强越野破障；坦克 300 经典越野圈层认可度高；山海 T2 轻度方盒子适合露营自驾。'
  },
  {
    id: 18,
    category: '家用SUV',
    destination: '15万级纯电SUV对比比亚迪元 Plus、埃安 AION Y 与奇瑞舒享家',
    topic: '关注网约车同款避坑、家用后排空间与网约车标签规避。',
    style: '家用舒适与性价比实用流',
    core: '结论：元 Plus 姿态时尚、海外爆款且刀片电池安全；AION Y 后排空间堪比大客厅但网约车标签重；舒享家全铝车身安全性高。'
  },
  {
    id: 19,
    category: '燃油豪华',
    destination: '20万豪华入门SUV对比凯迪拉克 XT4、沃尔沃 XC40 与雷克萨斯 UX',
    topic: '关注美系肌肉动力、北欧环保安全与日系免费保养。',
    style: '深度驾控体验与操纵性能流',
    core: '结论：XT4 2.0T+9AT动力同级无敌，美系静音极佳；XC40 环保无异味防碰撞主动安全；UX 适合女生单人代步省心。'
  },
  {
    id: 20,
    category: '代步紧凑',
    destination: '10-12万燃油小钢炮对比高尔夫 8、影豹与领克 03',
    topic: '关注排气声浪、改装潜力、弯道操控与青年首台车。',
    style: '深度驾控体验与操纵性能流',
    core: '结论：高尔夫 两厢情怀底盘扎实；领克 03 底盘调校最接近赛车，内饰高档；影豹 R 2.0T加速强劲声浪炸裂。'
  },
  {
    id: 21,
    category: '纯电新能源',
    destination: '18-22万纯电轿跑对比零跑 C01、零跑 C10 与小鹏 G6',
    topic: '关注全域800V、全场景扶摇架构、CTC电池一体化与性价比。',
    style: '极客硬核数据参数对比流',
    core: '结论：小鹏 G6 800V+XNGP智驾性价比之王；零跑 C10 全家舒享高配低价，质价比极高；C01 5米车长空间巨大。'
  },
  {
    id: 22,
    category: '插混增程',
    destination: '30万MPV对比腾势 D9、别克 GL8 PHEV 与岚图梦想家',
    topic: '关注商务接待、宜家宜商、二排航空座椅与碰撞安全。',
    style: '家用舒适与性价比实用流',
    core: '结论：腾势 D9 混动MPV销量冠军，保值率高；GL8 经典商务霸主底盘滤震极佳；梦想家 双电机四驱与空气悬架操控最好。'
  },
  {
    id: 23,
    category: '家用SUV',
    destination: '10-13万家用SUV对比哈弗 H6、长安 CS75 PLUS 与博越 L',
    topic: '关注国民神车对比、2.0T动力、爱信8AT变速箱稳定性。',
    style: '家用舒适与性价比实用流',
    core: '结论：CS75 PLUS 全系爱信8AT平顺可靠；H6 底盘沉稳皮实耐用；博越 L CMA架构操控灵活车机大屏前卫。'
  },
  {
    id: 24,
    category: '燃油豪华',
    destination: '80万高端豪华保时捷 Macan 对比 Cayenne 避坑配置选装指南',
    topic: '关注波尔多红内饰、Sport Chrono组件、PASM空气悬架选装。',
    style: '犀利毒舌车评与避坑指引流',
    core: '结论：Macan 2.0T提速够用但空间偏小；Cayenne 3.0T V6气场顶级保值。避坑提示：切勿在4S店选装超额昂贵轮毂，优先选装空气悬架。'
  },
  {
    id: 25,
    category: '代步紧凑',
    destination: '12万纯电跨界对比大众 ID.3 与 smart 精灵 #1',
    topic: '关注后驱底盘乐趣、转弯半径超小与时尚精品代步。',
    style: '深度驾控体验与操纵性能流',
    core: '结论：ID.3 转弯半径小得惊人，后驱底盘扎实有德系味；smart 精灵 #1 奔驰设计奢华前卫。预算到位选smart，实用选ID.3。'
  },
  {
    id: 26,
    category: '纯电新能源',
    destination: '50万顶级智能纯电对比高合 HiPhi Z、路特斯 Eletre 与蔚来 ET9',
    topic: '关注机甲外观、后轮转向、800V碳化硅与百万级底盘。',
    style: '极客硬核数据参数对比流',
    core: '结论：路特斯 Eletre 超跑级空气动力学与空气悬架操控无敌；蔚来 ET9 900V高压与主动悬架黑科技满满。'
  },
  {
    id: 27,
    category: '插混增程',
    destination: '15-20万家用增程对比零跑 C11 增程版与深蓝 S07',
    topic: '关注300公里纯电续航、可油可电、无电池焦虑与露营。',
    style: '家用舒适与性价比实用流',
    core: '结论：零跑 C11 前双叉臂后五连杆底盘堆料王，大电池纯电续航长；深蓝 S07 华为乾崑智驾加持，无框车门拉风。'
  },
  {
    id: 28,
    category: '家用SUV',
    destination: '20-25万纯电中型SUV对比特斯拉 Model Y 与比亚迪宋 L EV',
    topic: '关注悬架硬度改善、猎装轿跑姿态与后备箱容积。',
    style: '极客硬核数据参数对比流',
    core: '结论：Model Y 全球车王操控极致，辅助驾驶顺畅但悬架偏硬；宋 L CTB电池一体化与云辇-C底盘大幅改善舒适度。'
  },
  {
    id: 29,
    category: '燃油豪华',
    destination: '30万性能轿跑对比斯巴鲁 WRX、福特 Mustang 与本田 Type-R 避坑',
    topic: '关注水平对置发动机、全时四驱、手动挡情怀与进口关税。',
    style: '深度驾控体验与操纵性能流',
    core: '结论：Type-R 前驱王纯粹赛道利器；WRX 全时四驱机械抓地力极强；Mustang 2.3T外观拉风但后驱易滑。'
  },
  {
    id: 30,
    category: '代步紧凑',
    destination: '10-13万插混SUV对比奇瑞风云 T6、长城哈弗枭龙与吉利银河 L7 选购',
    topic: '关注插混系统平顺性、底盘防锈与冬季续航保值表现。',
    style: '家用舒适与性价比实用流',
    core: '结论：银河 L7 综合品质最均衡，车机交互好；枭龙 MAX 智能四驱Hi4适合雨雪路面；风云 T6 节能性价比突出。'
  }
];

const samples = ref<ChepingSample[]>(raw30Samples);

const filteredSamples = computed(() => {
  return samples.value.filter(s => {
    const matchCat = currentCategory.value === '全部' || s.category === currentCategory.value;
    const matchQuery = !searchQuery.value.trim() || 
      s.topic.includes(searchQuery.value) || 
      s.destination.includes(searchQuery.value) ||
      s.style.includes(searchQuery.value) || 
      s.core.includes(searchQuery.value);
    return matchCat && matchQuery;
  });
});

const totalPages = computed(() => Math.ceil(filteredSamples.value.length / pageSize) || 1);

const paginatedSamples = computed(() => {
  const start = (currentPage.value - 1) * pageSize;
  return filteredSamples.value.slice(start, start + pageSize);
});

watch([currentCategory, searchQuery], () => {
  currentPage.value = 1;
});
</script>
