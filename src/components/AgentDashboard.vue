<template>
  <section class="agent-dashboard">
    <div class="panel panel-left">
      <div class="input-panel">
        <h2>AI Agent 思考流程</h2>
        <p class="subtitle">輸入你的問題，觀察 Agent 怎麼一步一步思考。</p>
        <div class="mode-switch">
          <label for="agent-mode">Agent 模式</label>
          <div class="mode-control">
            <select id="agent-mode" v-model="selectedMode" @change="resetState">
              <option value="A">AiLineBOT 智能客服 Agent</option>
              <option value="B">百萬級廣告投放管理 Agent</option>
            </select>
          </div>
        </div>
        <div class="input-group">
          <textarea
            v-model="question"
            :placeholder="
              selectedMode === 'A'
                ? '請輸入問題，例如：蓮霧多少錢？'
                : '請輸入問題，例如：分析本月廣告投報率 (ROAS)'
            "
            rows="4"
          ></textarea>
          <button class="btn-send" @click="handleSubmit">送出問題</button>
        </div>

        <div class="quick-buttons">
          <button
            v-for="button in currentButtons"
            :key="button.label"
            @click="fillQuestion(button.question)"
          >
            {{ button.label }}
          </button>
        </div>
      </div>
    </div>

    <div class="panel panel-right">
      <div class="overview">
        <div class="timeline-card">
          <div class="panel-header">
            <h3>思考看板</h3>
            <span>Agent 展開內部流程</span>
          </div>
          <div class="timeline">
            <div
              v-for="(step, index) in steps"
              :key="index"
              class="timeline-step"
              :class="step.status"
            >
              <div class="step-index">Step {{ index + 1 }}</div>
              <div class="step-body">
                <div class="step-title">{{ step.title }}</div>
                <div class="step-description">{{ step.description }}</div>
              </div>
              <div class="step-status">
                <template v-if="step.status === 'success'">
                  <span class="icon success">✓</span>
                </template>
                <template v-else>
                  <span class="icon loading"></span>
                </template>
              </div>
            </div>
          </div>
        </div>

        <div class="answer-card">
          <div class="panel-header">
            <h3>最終回答卡片</h3>
            <span>AI Agent 為你整理的結果</span>
          </div>
          <div class="answer-content">
            <div class="answer-text" v-if="finalAnswer">{{ answerText }}</div>
            <div class="answer-placeholder" v-else>
              送出問題後，Agent 會依序分析意圖、選擇工具、取得資料並整理回答。
            </div>
          </div>
          <div class="result-block" v-if="toolResult">
            <div class="result-title">Tool 回傳資料</div>
            <pre><code>{{ toolResult }}</code></pre>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>

<script setup>
import { computed, reactive, ref } from 'vue'

const question = ref('')
const selectedMode = ref('A')
const steps = reactive([])
const toolResult = ref('')
const finalAnswer = ref('')
const answerText = ref('')
const isRunning = ref(false)

const agentModes = {
  A: {
    label: 'AiLineBOT 智能客服 Agent',
    buttons: [
      { label: '蓮霧多少錢？', question: '蓮霧多少錢？' },
      { label: '查詢訂單 A001', question: '查詢訂單 A001' },
      { label: '會有農藥殘留嗎？', question: '會有農藥殘留嗎？' }
    ]
  },
  B: {
    label: '百萬級廣告投放管理 Agent',
    buttons: [
      { label: '分析本月廣告投報率 (ROAS)', question: '分析本月廣告投報率 (ROAS)' },
      { label: '自動調整 FB/LINE 預算配比', question: '自動調整 FB/LINE 預算配比' },
      { label: '產出下週生鮮水果文案', question: '產出下週生鮮水果文案' }
    ]
  }
}

const currentButtons = computed(() => agentModes[selectedMode.value].buttons)

const mockTools = {
  getProducts() {
    return [
      { name: '【無毒農】有機黑糖芭比蓮霧', price: 1280, stock: 15 },
      { name: '【自建食安實驗室】質譜快篩有機蔬菜箱', price: 699, stock: 50 }
    ]
  },
  getOrder(orderId) {
    return {
      id: orderId || 'A001',
      status: '已通過微生物檢驗，出貨中',
      lab_check: '質譜快篩：未檢出農藥殘留'
    }
  },
  searchFAQ(keyword) {
    return {
      question: '你們的蔬菜有農藥殘留嗎？',
      answer:
        '無毒農擁有自建食安實驗室，具備質譜快篩與微生物檢驗能力，做到每天逐批檢驗，合格才出貨，請安心食用！'
    }
  },
  analyzeROAS() {
    return {
      campaign: '本月生鮮廣告投放',
      impressions: 128000,
      clicks: 9600,
      spend: 19800,
      revenue: 63400,
      roas: 3.2,
      topChannel: 'Facebook',
      trend: 'LINE 成本效益上升，建議調整預算配置。'
    }
  },
  adjustBudget() {
    return {
      current: { FB: 55, LINE: 45 },
      suggested: { FB: 48, LINE: 52 },
      rationale: '依據近期投放數據，LINE 的成本轉換率優於 FB，建議微調預算配比以提升整體 ROAS。'
    }
  },
  generateCopy() {
    return {
      title: '甜美蓮霧大促｜直送無毒鮮果',
      body: '本週獨家生鮮水果特賣，嚴選無毒蓮霧與頂級水果，立即下單享限時優惠。FB/LINE 廣告文案已優化為品牌清新語氣，提升點閱與轉換。'
    }
  }
}

const fillQuestion = (value) => {
  question.value = value
  handleSubmit()
}

const resetState = () => {
  steps.length = 0
  toolResult.value = ''
  finalAnswer.value = ''
  answerText.value = ''
  isRunning.value = false
}

const determineTool = (raw) => {
  const text = raw.toLowerCase()
  if (selectedMode.value === 'B') {
    if (text.includes('roas') || text.includes('廣告投報率') || text.includes('投報率')) {
      return { name: 'analyzeROAS', args: [] }
    }
    if (text.includes('預算') || text.includes('配比')) {
      return { name: 'adjustBudget', args: [] }
    }
    if (text.includes('文案') || text.includes('水果') || text.includes('廣告')) {
      return { name: 'generateCopy', args: [] }
    }
    return { name: 'analyzeROAS', args: [] }
  }

  if (text.includes('a001') || text.includes('查詢訂單') || text.includes('訂單')) {
    return { name: 'getOrder', args: ['A001'] }
  }
  if (text.includes('農藥') || text.includes('殘留') || text.includes('會有農藥')) {
    return { name: 'searchFAQ', args: ['你們的蔬菜有農藥殘留嗎？'] }
  }
  if (
    text.includes('多少錢') ||
    text.includes('蓮霧') ||
    text.includes('黑糖芭比蓮霧') ||
    text.includes('有機蔬菜箱')
  ) {
    return { name: 'getProducts', args: [] }
  }
  return { name: 'getProducts', args: [] }
}

const generateAnswer = (questionText, toolName, data) => {
  if (toolName === 'getOrder') {
    return `根據訂單 ${data.id} 的查詢，狀態為「${data.status}」。`
  }
  if (toolName === 'searchFAQ') {
    return `FAQ：${data.question}，建議：${data.answer}`
  }
  if (toolName === 'getProducts') {
    const product = data[0]
    return `找到商品「${product.name}」，售價 ${product.price} 元，庫存剩餘 ${product.stock} 件。`
  }
  if (toolName === 'analyzeROAS') {
    return `本月廣告 ROAS 為 ${data.roas}，預估營收 ${data.revenue} 元，主要投放於 ${data.topChannel}，策略建議：${data.trend}`
  }
  if (toolName === 'adjustBudget') {
    return `依據目前 FB/LINE 投放表現，建議將預算從 FB ${data.current.FB}% / LINE ${data.current.LINE}% 調整為 FB ${data.suggested.FB}% / LINE ${data.suggested.LINE}%。理由：${data.rationale}`
  }
  if (toolName === 'generateCopy') {
    return `已產出下週生鮮水果廣告文案：\n標題：${data.title}\n內容：${data.body}`
  }
  return '已完成查詢，請查看結果。'
}

const pushStep = (title, description) => {
  steps.push({ title, description, status: 'pending' })
}

const completeStep = (index) => {
  if (steps[index]) {
    steps[index].status = 'success'
  }
}

const animateFinalAnswer = (text) => {
  answerText.value = ''
  let idx = 0
  const intervalId = setInterval(() => {
    answerText.value += text[idx] || ''
    idx += 1
    if (idx >= text.length) {
      clearInterval(intervalId)
    }
  }, 40)
}

const handleSubmit = () => {
  if (!question.value.trim() || isRunning.value) return
  resetState()
  isRunning.value = true

  const query = question.value.trim()
  const isAdMode = selectedMode.value === 'B'
  const stepDescriptions = isAdMode
    ? [
        '分析使用者廣告需求與投放目標，判定為廣告投放管理方案。',
        '根據目標與預算路由到最佳分析與優化工具。',
        '成功獲取廣告投放報表與預算配置建議。',
        '整合廣告策略與投放規範，生成優化建議與執行方案。'
      ]
    : [
        '分析使用者 Context 中的意圖，成功判定為「生鮮電商業務與食安查詢」。',
        '根據上下文 Context 評估，決定呼叫對應的 Mock Tool 進行數據檢索。',
        '成功獲取無毒農資料庫與食安實驗室的 Mock JSON 數據。',
        '結合品牌知識庫與客服 Workflow 規範，生成符合品牌語氣的最終回覆。'
      ]

  pushStep('[意圖分析] 啟動 Intent-Classifier Agent', stepDescriptions[0])
  pushStep('[工具路由] Router Agent 分流決策', stepDescriptions[1])
  pushStep('[數據檢索] 執行 Tool Call 撈取後台資料', stepDescriptions[2])
  pushStep('[答案生成] 整合 RAG 知識庫與 LLM 輸出', stepDescriptions[3])

  const selection = determineTool(query)
  const toolName = selection.name
  const toolArgs = selection.args

  setTimeout(() => {
    completeStep(0)
    steps[0].description = stepDescriptions[0]
  }, 800)

  setTimeout(() => {
    completeStep(1)
    steps[1].description = `根據上下文 Context 評估，決定呼叫 ${toolName}(${toolArgs.map((arg) => `"${arg}"`).join(', ')}) 進行數據檢索。`
  }, 1600)

  setTimeout(() => {
    completeStep(2)
    const result = mockTools[toolName](...toolArgs)
    toolResult.value = JSON.stringify(result, null, 2)
    steps[2].description = stepDescriptions[2]
  }, 2400)

  setTimeout(() => {
    completeStep(3)
    finalAnswer.value = generateAnswer(query, toolName, JSON.parse(toolResult.value))
    animateFinalAnswer(finalAnswer.value)
    steps[3].description = stepDescriptions[3]
    isRunning.value = false
  }, 3200)
}
</script>

<style lang="scss" scoped>
$bg-dark: #1a1e29;
$cyber-cyan: #00f2fe;
$theme-blue: #0072ff;
$panel-bg: #202533;
$text-light: #edf2ff;
$code-bg: rgba(5, 15, 35, 0.95);

.agent-dashboard {
  display: grid;
  grid-template-columns: 1fr 1.4fr;
  gap: 24px;
  padding: 24px;
  min-height: 100vh;
  background: linear-gradient(135deg, #11141f 0%, #181d2a 45%, #121624 100%);
  color: $text-light;

  .panel {
    border-radius: 24px;
    padding: 24px;
    background: rgba(10, 16, 28, 0.95);
    box-shadow: 0 24px 70px rgba(0, 0, 0, 0.35);
  }

  .panel-left {
    display: flex;
    align-items: flex-start;

    .input-panel {
      width: 100%;

      h2 {
        margin-bottom: 10px;
        font-size: 1.75rem;
        letter-spacing: 0.05em;
        background: linear-gradient(90deg, $cyber-cyan, $theme-blue);
        -webkit-background-clip: text;
        color: transparent;
      }

      .subtitle {
        margin-bottom: 24px;
        opacity: 0.78;
        line-height: 1.75;
      }

      .mode-switch {
        margin-bottom: 18px;
        display: grid;
        gap: 10px;

        label {
          color: rgba(255, 255, 255, 0.72);
          font-size: 0.95rem;
          letter-spacing: 0.05em;
        }

        .mode-control {
          position: relative;

          select {
            width: 100%;
            padding: 14px 18px;
            border-radius: 18px;
            border: 1px solid rgba(255, 255, 255, 0.14);
            background: rgba(5, 10, 20, 0.95);
            color: $text-light;
            outline: none;
            font-size: 1rem;
            appearance: none;
            cursor: pointer;
          }

          &::after {
            content: '▾';
            position: absolute;
            right: 18px;
            top: 50%;
            transform: translateY(-50%);
            color: $cyber-cyan;
            pointer-events: none;
            font-size: 0.95rem;
          }
        }
      }

      .input-group {
        display: grid;
        gap: 16px;

        textarea {
          width: 100%;
          border: 1px solid rgba(255, 255, 255, 0.08);
          border-radius: 18px;
          background: rgba(10, 18, 34, 0.96);
          color: $text-light;
          padding: 18px;
          resize: none;
          font-size: 1rem;
          min-height: 140px;
          outline: none;
          transition: border-color 0.2s ease;

          &:focus {
            border-color: rgba($cyber-cyan, 0.65);
            box-shadow: 0 0 0 4px rgba(0, 242, 254, 0.08);
          }
        }

        .btn-send {
          align-self: flex-end;
          justify-self: flex-end;
          padding: 0.95rem 1.7rem;
          border-radius: 999px;
          border: none;
          background: linear-gradient(135deg, $cyber-cyan, $theme-blue);
          color: #071827;
          font-weight: 700;
          cursor: pointer;
          transition:
            transform 0.2s ease,
            filter 0.2s ease;

          &:hover {
            transform: translateY(-1px);
            filter: brightness(1.05);
          }
        }
      }

      .quick-buttons {
        margin-top: 26px;
        display: grid;
        gap: 12px;

        button {
          width: 100%;
          border: 1px solid rgba(255, 255, 255, 0.08);
          border-radius: 14px;
          background: rgba(255, 255, 255, 0.03);
          color: $text-light;
          padding: 14px 16px;
          cursor: pointer;
          transition:
            background 0.2s ease,
            transform 0.2s ease;

          &:hover {
            background: rgba(0, 242, 254, 0.08);
            transform: translateX(4px);
          }
        }
      }
    }
  }

  .panel-right {
    .overview {
      display: flex;
      flex-direction: column;
      gap: 20px;

      .panel-header {
        display: flex;
        justify-content: space-between;
        align-items: center;
        margin-bottom: 18px;

        h3 {
          margin: 0;
          font-size: 1.2rem;
          letter-spacing: 0.06em;
        }

        span {
          opacity: 0.7;
          font-size: 0.95rem;
        }
      }

      .timeline-card,
      .answer-card {
        background: $panel-bg;
        border: 1px solid rgba(255, 255, 255, 0.08);
        border-radius: 22px;
        padding: 22px;
      }

      .timeline {
        display: grid;
        gap: 16px;

        .timeline-step {
          display: grid;
          grid-template-columns: auto 1fr auto;
          gap: 14px;
          padding: 18px;
          border-radius: 18px;
          background: rgba(8, 14, 26, 0.92);
          border: 1px solid rgba(255, 255, 255, 0.05);
          transform: translateX(-12px);
          opacity: 0;
          animation: fadeInUp 0.6s forwards;

          &.success {
            border-color: rgba(0, 242, 254, 0.25);
          }

          .step-index {
            display: inline-flex;
            align-items: center;
            justify-content: center;
            width: 44px;
            height: 44px;
            border-radius: 14px;
            background: rgba(0, 242, 254, 0.08);
            color: $cyber-cyan;
            font-weight: 700;
            font-size: 0.95rem;
          }

          .step-body {
            display: grid;
            gap: 6px;

            .step-title {
              color: $text-light;
              font-weight: 700;
            }

            .step-description {
              opacity: 0.7;
              line-height: 1.5;
              font-size: 0.95rem;
            }
          }

          .step-status {
            display: flex;
            align-items: center;
          }

          .icon {
            width: 28px;
            height: 28px;
            border-radius: 50%;
            display: inline-flex;
            align-items: center;
            justify-content: center;
            font-weight: 700;
            color: #fff;
          }

          .loading {
            border: 3px solid rgba(255, 255, 255, 0.1);
            border-top-color: $cyber-cyan;
            animation: spin 1s linear infinite;
            background: transparent;
          }

          .success {
            background: rgba(0, 255, 213, 0.18);
            color: #00ffdc;
          }
        }
      }

      .answer-card {
        display: grid;
        gap: 18px;

        .answer-content {
          min-height: 120px;
          padding: 18px;
          border-radius: 18px;
          background: rgba(5, 10, 20, 0.95);
          border: 1px solid rgba(255, 255, 255, 0.06);
          display: flex;
          align-items: center;
        }

        .answer-text {
          color: $text-light;
          line-height: 1.8;
          font-size: 1rem;
          white-space: pre-wrap;
          letter-spacing: 0.02em;
        }

        .answer-placeholder {
          color: rgba(255, 255, 255, 0.6);
          line-height: 1.7;
        }

        .result-block {
          border-radius: 18px;
          background: $code-bg;
          padding: 16px;
          border: 1px solid rgba(0, 122, 255, 0.15);

          .result-title {
            margin-bottom: 10px;
            font-weight: 700;
            color: $cyber-cyan;
          }

          pre {
            margin: 0;
            overflow-x: auto;
            color: #d7e9ff;
            font-size: 0.95rem;
            line-height: 1.6;
          }

          code {
            display: block;
            white-space: pre-wrap;
          }
        }
      }
    }
  }
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}

@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}
</style>
