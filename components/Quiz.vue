<template>
  <div class="quiz-container">
    <div class="quiz-card">
      <!-- Шапка с заголовком -->
      <div class="quiz-header">
        <h1>История промышленности и торговли России</h1>
        <div class="progress-info" v-if="!isFinished">
          Вопрос {{ currentQuestionIndex + 1 }} из {{ questions.length }}
        </div>
      </div>

      <div v-if="isFinished" class="result-screen">
        <h2>Тест завершен!</h2>
        <div class="result-score">
          Правильных ответов: <strong>{{ correctAnswers }}</strong> из
          {{ questions.length }}
        </div>
        <div class="score-circle" :class="getScoreClass">
          {{ Math.round((correctAnswers / questions.length) * 100) }}%
        </div>
        <p class="result-message" v-html="getResultMessage"></p>
        
        <!-- Кнопки управления -->
        <div class="result-buttons">
          <button @click="restartQuiz" class="restart-btn">Пройти заново</button>
          <button @click="clearProgress" class="clear-btn">Очистить историю</button>
        </div>
        
        <!-- Статистика -->
        <div class="statistics" v-if="savedResults.length > 0">
          <h3>Предыдущие результаты:</h3>
          <div class="stats-list">
            <div v-for="(result, idx) in savedResults" :key="idx" class="stat-item">
              <span class="stat-date">{{ formatDate(result.date) }}</span>
              <span class="stat-score">{{ result.score }}/{{ result.total }}</span>
              <span class="stat-percent">{{ Math.round((result.score/result.total)*100) }}%</span>
            </div>
          </div>
        </div>
      </div>

      <!-- Экран вопроса -->
      <div v-else class="question-screen">
        <!-- Прогресс бар -->
        <div class="progress-bar">
          <div
            class="progress-fill"
            :style="{ width: progressPercentage + '%' }"
          ></div>
        </div>

        <h2 class="question-text">{{ currentQuestion.text }}</h2>

        <!-- Варианты ответов -->
        <div class="options-list">
          <label
            v-for="(option, idx) in currentQuestion.options"
            :key="idx"
            class="option-item"
            :class="{ 'option-selected': selectedAnswer === idx }"
          >
            <input
              type="radio"
              :name="'question_' + currentQuestionIndex"
              :value="idx"
              v-model="selectedAnswer"
            />
            <span class="option-marker">{{ getLetter(idx) }}.</span>
            <span class="option-text">{{ option }}</span>
          </label>
        </div>

        <!-- Навигационные кнопки -->
        <div class="navigation-buttons">
          <button
            @click="prevQuestion"
            :disabled="currentQuestionIndex === 0"
            class="nav-btn prev-btn"
          >
            ← Назад
          </button>
          <button
            @click="nextQuestion"
            :disabled="selectedAnswer === null"
            class="nav-btn next-btn"
          >
            {{ isLastQuestion ? 'Завершить' : 'Далее →' }}
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'HistoryQuiz',
  data() {
    return {
      // Вопросы из документа с правильными ответами
      questions: [
        {
          text: 'В каком году было учреждено Министерство торговли и промышленности Российской империи?',
          options: ['1907', '1905', '1917'],
          correct: 1, // 1905 год (индекс 1)
        },
        {
          text: 'Первым министром промышленности и торговли стал:',
          options: ['В.И. Тимирязев', 'М.М. Федоров', 'С.Ю. Витте'],
          correct: 0, // В.И. Тимирязев (индекс 0)
        },
        {
          text: 'Ныне действующим министром промышленности и торговли Российской Федерации является:',
          options: ['Д.В. Мантуров', 'А.А. Алиханов', 'С.В. Лавров'],
          correct: 1, // А.А. Алиханов (индекс 1)
        },
        {
          text: 'Назовите имя создателя и главного конструктора танка Т-34:',
          options: ['Михаил Калашников', 'Михаил Кошкин', 'Сергей Королёв'],
          correct: 1, // Михаил Кошкин (индекс 1)
        },
        {
          text: 'Как называется российская гиперзвуковая ракета, которая является самой совершенной ракетой в мире, против которой нет способов противодействия? Она способна лететь со скоростью, превышающей скорость звука в 12 раз.',
          options: ['Х-47М2 «Кинжал»', 'И-48Т4 «Сабля»', 'М-61Р8 «Рапира»'],
          correct: 0, // Х-47М2 «Кинжал» (индекс 0)
        },
        {
          text: 'Где находится Космодром «Восточный»?',
          options: ['Астраханская область', 'Забайкальский край', 'Амурская область'],
          correct: 2, // Амурская область (индекс 2)
        },
        {
          text: 'Какая страна обладает единственным в мире атомным ледокольным флотом, фактически являясь хозяйкой в Арктике?',
          options: ['Россия', 'Беларусь', 'США'],
          correct: 0, // Россия (индекс 0)
        },
        {
          text: 'Какие Российские системы ПВО считаются лучшими в мире?',
          options: ['Д-300 и Д-700', 'С-400 и С-500', 'К-200 и К-400'],
          correct: 1, // С-400 и С-500 (индекс 1)
        },
        {
          text: 'Каким ракетным комплексом пятого поколения вооружены первые в мире стратегические атомные подводные лодки четвертого поколения?',
          options: ['«Молот»', '«Болас»', '«Булава»'],
          correct: 2, // «Булава» (индекс 2)
        },
        {
          text: 'Первый российский газотурбинный агрегат морского применения мощностью 8 МВт, способный заместить зарубежное оборудование в шельфовом сегменте:',
          options: ['ГТА-М9', 'ГТА-8', 'ГТА-Б8'],
          correct: 1, // ГТА-8 (индекс 1)
        },
      ],
      currentQuestionIndex: 0,
      userAnswers: [],
      isFinished: false,
      savedResults: [], // Массив сохраненных результатов
    };
  },
  
  // При создании компонента загружаем данные из localStorage
  created() {
    this.loadFromStorage();
  },
  
  computed: {
    currentQuestion() {
      return this.questions[this.currentQuestionIndex];
    },
    selectedAnswer: {
      get() {
        return this.userAnswers[this.currentQuestionIndex] ?? null;
      },
      set(value) {
        this.userAnswers[this.currentQuestionIndex] = value;
        this.saveToStorage(); // Сохраняем при каждом выборе ответа
      },
    },
    progressPercentage() {
      return ((this.currentQuestionIndex + 1) / this.questions.length) * 100;
    },
    isLastQuestion() {
      return this.currentQuestionIndex === this.questions.length - 1;
    },
    correctAnswers() {
      let count = 0;
      this.questions.forEach((question, index) => {
        if (this.userAnswers[index] === question.correct) {
          count++;
        }
      });
      return count;
    },
    getScoreClass() {
      const percentage = (this.correctAnswers / this.questions.length) * 100;
      if (percentage >= 80) return 'score-excellent';
      if (percentage >= 60) return 'score-good';
      if (percentage >= 40) return 'score-average';
      return 'score-poor';
    },
    getResultMessage() {
      const percentage = (this.correctAnswers / this.questions.length) * 100;
      if (percentage === 100) {
        return '🎉 Поздравляем! Вы блестяще знаете историю промышленности и торговли России!';
      }
      if (percentage >= 80) {
        return '🌟 Отличный результат! Вы хорошо разбираетесь в теме.';
      }
      if (percentage >= 60) {
        return '📚 Хороший результат! Но есть куда стремиться.';
      }
      if (percentage >= 40) {
        return '📖 Средний результат. Рекомендуем повторить материал.';
      }
      return '💪 Не расстраивайтесь! Попробуйте пройти тест еще раз.';
    },
  },
  
  watch: {
    // Следим за изменением индекса вопроса и сохраняем
    currentQuestionIndex() {
      this.saveToStorage();
    },
    // Следим за завершением теста
    isFinished(newVal) {
      if (newVal) {
        this.saveResult(); // Сохраняем результат в историю
      }
      this.saveToStorage();
    }
  },
  
  methods: {
    // Сохранение в localStorage
    saveToStorage() {
      const quizState = {
        currentQuestionIndex: this.currentQuestionIndex,
        userAnswers: this.userAnswers,
        isFinished: this.isFinished,
        savedResults: this.savedResults
      };
      localStorage.setItem('quizHistoryState', JSON.stringify(quizState));
    },
    
    // Загрузка из localStorage
    loadFromStorage() {
      const saved = localStorage.getItem('quizHistoryState');
      if (saved) {
        try {
          const state = JSON.parse(saved);
          this.currentQuestionIndex = state.currentQuestionIndex || 0;
          this.userAnswers = state.userAnswers || [];
          this.isFinished = state.isFinished || false;
          this.savedResults = state.savedResults || [];
        } catch (e) {
          console.error('Ошибка загрузки из localStorage:', e);
        }
      }
    },
    
    // Сохранение результата в историю
    saveResult() {
      const result = {
        date: new Date().toISOString(),
        score: this.correctAnswers,
        total: this.questions.length
      };
      
      this.savedResults.unshift(result); // Добавляем в начало массива
      
      // Ограничим историю 10 последними результатами
      if (this.savedResults.length > 10) {
        this.savedResults = this.savedResults.slice(0, 10);
      }
      
      this.saveToStorage();
    },
    
    // Форматирование даты
    formatDate(dateString) {
      const date = new Date(dateString);
      return date.toLocaleString('ru-RU', {
        day: '2-digit',
        month: '2-digit',
        year: 'numeric',
        hour: '2-digit',
        minute: '2-digit'
      });
    },
    
    // Очистка всей истории
    clearProgress() {
      if (confirm('Вы уверены, что хотите очистить всю историю результатов?')) {
        this.savedResults = [];
        localStorage.removeItem('quizHistoryState');
        this.restartQuiz();
      }
    },
    
    nextQuestion() {
      if (this.isLastQuestion) {
        this.isFinished = true;
      } else {
        this.currentQuestionIndex++;
      }
    },
    
    prevQuestion() {
      if (this.currentQuestionIndex > 0) {
        this.currentQuestionIndex--;
      }
    },
    
    restartQuiz() {
      this.currentQuestionIndex = 0;
      this.userAnswers = [];
      this.isFinished = false;
      this.saveToStorage();
    },
    
    getLetter(index) {
      const letters = ['а', 'б', 'в', 'г', 'д', 'е'];
      return letters[index] || index + 1;
    },
  },
};
</script>

<style scoped>
.quiz-container {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  background: linear-gradient(135deg, #1a3650 0%, #2c5282 100%);
  font-family: 'Arial', sans-serif;
  padding: 16px;
  box-sizing: border-box;
}

.quiz-card {
  background: white;
  border-radius: 20px;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
  width: 100%;
  max-width: 700px;
  overflow: hidden;
  animation: slide-up 0.4s ease;
}

@keyframes slide-up {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.quiz-header {
  padding: 24px 30px 10px 30px;
  display: flex;
  justify-content: space-between;
  align-items: baseline;
  border-bottom: 2px solid #e2e8f0;
  background: linear-gradient(90deg, #f7fafc, #ffffff);
}

.quiz-header h1 {
  margin: 0;
  font-size: 1.5rem;
  color: #1a3650;
  font-weight: 600;
}

.progress-info {
  font-size: 1rem;
  color: #2c5282;
  font-weight: 600;
  background: #ebf8ff;
  padding: 6px 12px;
  border-radius: 20px;
}

.question-screen {
  padding: 20px 30px 30px 30px;
}

.progress-bar {
  height: 8px;
  background-color: #e2e8f0;
  border-radius: 10px;
  margin-bottom: 25px;
  overflow: hidden;
}

.progress-fill {
  height: 100%;
  background: linear-gradient(90deg, #2c5282, #4299e1);
  border-radius: 10px;
  transition: width 0.3s ease;
}

.question-text {
  font-size: 1.3rem;
  color: #1a3650;
  margin: 0 0 25px 0;
  line-height: 1.5;
  font-weight: 500;
}

.options-list {
  display: flex;
  flex-direction: column;
  gap: 12px;
  margin-bottom: 35px;
}

.option-item {
  display: flex;
  align-items: center;
  padding: 16px 20px;
  background-color: #f7fafc;
  border: 2px solid transparent;
  border-radius: 12px;
  cursor: pointer;
  transition: all 0.2s ease;
}

.option-item:hover {
  background-color: #edf2f7;
  transform: translateX(5px);
}

.option-selected {
  background-color: #ebf8ff;
  border-color: #2c5282;
  box-shadow: 0 4px 12px rgba(44, 82, 130, 0.1);
}

.option-item input[type='radio'] {
  appearance: none;
  -webkit-appearance: none;
  width: 22px;
  height: 22px;
  border: 2px solid #cbd5e0;
  border-radius: 50%;
  margin-right: 15px;
  position: relative;
  cursor: pointer;
  transition: all 0.2s;
}

.option-item input[type='radio']:checked {
  border-color: #2c5282;
  background-color: #2c5282;
  box-shadow: inset 0 0 0 5px white;
}

.option-marker {
  font-weight: 700;
  color: #2c5282;
  margin-right: 10px;
  min-width: 25px;
  font-size: 1.1rem;
}

.option-text {
  font-size: 1.05rem;
  color: #2d3748;
  line-height: 1.4;
}

.navigation-buttons {
  display: flex;
  justify-content: space-between;
  gap: 15px;
}

.nav-btn {
  padding: 14px 35px;
  border: none;
  border-radius: 50px;
  font-size: 1.1rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s ease;
  flex: 1;
  letter-spacing: 0.5px;
}

.prev-btn {
  background-color: #e2e8f0;
  color: #1a3650;
}

.prev-btn:hover:not(:disabled) {
  background-color: #cbd5e0;
}

.next-btn {
  background: linear-gradient(135deg, #2c5282, #4299e1);
  color: white;
}

.next-btn:hover:not(:disabled) {
  transform: translateX(3px);
  box-shadow: 0 7px 14px rgba(44, 82, 130, 0.4);
}

.nav-btn:disabled {
  opacity: 0.4;
  cursor: not-allowed;
  pointer-events: none;
}

.result-screen {
  padding: 40px 30px;
  text-align: center;
  background: linear-gradient(135deg, #f7fafc, #ffffff);
}

.result-screen h2 {
  color: #1a3650;
  font-size: 2.2rem;
  margin: 0 0 20px 0;
  font-weight: 700;
}

.result-score {
  font-size: 1.5rem;
  color: #4a5568;
  margin-bottom: 25px;
}

.result-score strong {
  color: #2c5282;
  font-size: 2rem;
}

.score-circle {
  width: 150px;
  height: 150px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  margin: 0 auto 30px;
  font-size: 2.5rem;
  font-weight: 700;
  color: white;
  background: #2c5282;
  box-shadow: 0 10px 20px rgba(0, 0, 0, 0.2);
}

.score-excellent {
  background: linear-gradient(135deg, #2ecc71, #27ae60);
}

.score-good {
  background: linear-gradient(135deg, #3498db, #2980b9);
}

.score-average {
  background: linear-gradient(135deg, #f39c12, #e67e22);
}

.score-poor {
  background: linear-gradient(135deg, #e74c3c, #c0392b);
}

.result-message {
  font-size: 1.2rem;
  color: #4a5568;
  margin-bottom: 35px;
  line-height: 1.6;
  padding: 0 20px;
}

.result-buttons {
  display: flex;
  gap: 15px;
  justify-content: center;
  margin-bottom: 30px;
  flex-wrap: wrap;
}

.restart-btn {
  background: linear-gradient(135deg, #2c5282, #4299e1);
  color: white;
  border: none;
  padding: 16px 35px;
  border-radius: 50px;
  font-size: 1.1rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  border: 2px solid transparent;
  min-width: 200px;
}

.restart-btn:hover {
  transform: scale(1.05);
  box-shadow: 0 15px 30px rgba(44, 82, 130, 0.4);
}

.clear-btn {
  background: white;
  color: #2c5282;
  border: 2px solid #2c5282;
  padding: 16px 35px;
  border-radius: 50px;
  font-size: 1.1rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  min-width: 200px;
}

.clear-btn:hover {
  background: #2c5282;
  color: white;
}

.statistics {
  margin-top: 30px;
  padding-top: 20px;
  border-top: 2px solid #e2e8f0;
}

.statistics h3 {
  color: #1a3650;
  margin-bottom: 15px;
  font-size: 1.2rem;
}

.stats-list {
  max-height: 200px;
  overflow-y: auto;
  padding: 5px;
}

.stat-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px 15px;
  background: #f7fafc;
  border-radius: 10px;
  margin-bottom: 8px;
  transition: all 0.2s ease;
}

.stat-item:hover {
  background: #edf2f7;
  transform: translateX(5px);
}

.stat-date {
  font-size: 0.9rem;
  color: #718096;
}

.stat-score {
  font-weight: 600;
  color: #2c5282;
}

.stat-percent {
  background: #2c5282;
  color: white;
  padding: 4px 8px;
  border-radius: 20px;
  font-size: 0.85rem;
  font-weight: 600;
}

@media (max-width: 600px) {
  .quiz-header {
    padding: 20px 20px 10px 20px;
    flex-direction: column;
    gap: 12px;
    align-items: flex-start;
  }

  .quiz-header h1 {
    font-size: 1.3rem;
  }

  .question-screen,
  .result-screen {
    padding: 15px 20px 25px 20px;
  }

  .question-text {
    font-size: 1.1rem;
  }

  .option-item {
    padding: 14px 16px;
  }

  .option-text {
    font-size: 0.95rem;
  }

  .navigation-buttons {
    flex-direction: column-reverse;
  }

  .nav-btn {
    width: 100%;
    padding: 14px 20px;
  }

  .score-circle {
    width: 120px;
    height: 120px;
    font-size: 2rem;
  }

  .result-buttons {
    flex-direction: column;
  }

  .restart-btn,
  .clear-btn {
    width: 100%;
    min-width: auto;
  }
}
</style>