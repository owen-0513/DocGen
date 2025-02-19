<template>
  <div class="background d-flex align-items-center justify-content-center">
    <div class="container-fluid">
      <div class="row justify-content-center">
        <!-- 左側: 提問輸入框 -->
        <div class="col-lg-5 col-md-6 col-sm-10">
          <div class="card shadow-lg p-4">
            <h2 class="text-primary fw-bold">
              <i class="bi bi-chat-dots"></i> 問答系統
            </h2>
            <p class="text-muted">輸入你的問題，會回應並提供下載 Word 文檔下載功能。</p>

            <textarea class="form-control" v-model="question" placeholder="請輸入你的問題..." rows="4"></textarea>

            <button class="btn btn-primary mt-3 w-100" @click="askAI" :disabled="loading">
              <span v-if="loading"><i class="bi bi-hourglass-split"></i> 回答中...</span>
              <span v-else><i class="bi bi-send"></i> 提交問題</span>
            </button>

            <p v-if="status" :class="statusClass" class="mt-3 text-center">{{ status }}</p>
          </div>
        </div>

        <!-- 右側: AI 回應 -->
        <div class="col-lg-5 col-md-6 col-sm-10 mt-3 mt-md-0">
          <div class="card shadow-lg p-4">
            <h3 class="text-success fw-bold">
              <i class="bi bi-file-text"></i> 回應
            </h3>
            <div v-if="answer && !isError" class="border rounded p-3 bg-light text-dark" style="min-height: 150px;">
              <p v-html="answer"></p>
              <button class="btn btn-success w-100 mt-3" @click="downloadWord">
                <i class="bi bi-download"></i> 確認後下載 Word
              </button>
            </div>
            <p v-if="isError" class="text-danger">⚠️ {{ answer }}</p>
            <p v-else-if="!answer" class="text-muted">請輸入問題，在這裡會顯示回答...</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      question: "",
      answer: "",
      status: "",
      isError: false,
      loading: false,
    };
  },
  computed: {
    statusClass() {
      return this.isError ? "text-danger" : "text-success";
    },
  },
  methods: {
    async askAI() {
      if (!this.question.trim()) {
        this.status = "⚠️ 請輸入問題！";
        this.isError = true;
        return;
      }

      this.status = "⏳ 正在生成回答，請稍候...";
      this.isError = false;
      this.loading = true;
      this.answer = ""; 

      try {
        const API_URL = import.meta.env.VITE_API_URL || "http://127.0.0.1:5000";

        const response = await axios.post(`${API_URL}/askQuestion`, { question: this.question });

        console.log("🔍 API 回應內容：", response.data);

        if (response.data.error) {
          this.answer = `⚠️ 錯誤：${response.data.error}`;
          this.isError = true;
        } else {
          this.answer = response.data.answer;  
          this.status = "✅ 回應完成！";
        }
      } catch (error) {
        console.error("❌ API 發生錯誤:", error);
        this.status = `❌ 發生錯誤: ${error.message}`;
        this.answer = "❌ 無法連接伺服器，請稍後再試。";
        this.isError = true;
      } finally {
        this.loading = false;
      }
    },

    async downloadWord() {
      if (!this.answer || this.isError) {
        this.status = "⚠️ 無法下載 Word，請先獲取 AI 回應。";
        return;
      }

      try {
        const API_URL = import.meta.env.VITE_API_URL || "http://127.0.0.1:5000";

        const response = await axios.post(
          `${API_URL}/askQuestion`,
          { question: this.question, answer: this.answer, download: true }, 
          { responseType: "blob" }
        );

        const blob = new Blob([response.data], {
          type: "application/vnd.openxmlformats-officedocument.wordprocessingml.document"
        });
        const url = window.URL.createObjectURL(blob);
        const link = document.createElement("a");
        link.href = url;
        link.setAttribute("download", "簡易回答.docx");
        document.body.appendChild(link);
        link.click();
        document.body.removeChild(link);

        this.status = "✅ 下載成功！";
      } catch (error) {
        console.error("❌ 下載 Word 錯誤:", error);
        this.status = `❌ 發生錯誤: ${error.message}`;
        this.isError = true;
      }
    },
  },
  mounted() {
    document.title = "問答系統 - 即時 AI 回應 & 下載 Word";
  }
};
</script>

<style>
/* 背景設計 */
.background {
  background: linear-gradient(135deg, #6e8efb, #a777e3);
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  min-height: 100vh;
  overflow: hidden;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 20px;
}



/* 修正 container 對齊問題 */
.container-fluid {
  width: 90%;
  max-width: 1200px;
  padding: 0 20px;
}


/* 讓 row 內的 col-* 保持對齊 */
.row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 20px;
  width: 100%;
}


/* 確保卡片不會太窄 */
.card {
  width: 100%;
  min-height: 200px;
}

/* 修正小螢幕適應 */
@media (max-width: 768px) {
  .row {
    flex-direction: column;
    align-items: center;
  }

  .col-md-6, .col-lg-5 {
    width: 100%;
    max-width: 500px;
    text-align: center;
  }

  .card {
    width: 100%;
  }
}

</style>
