<template>
  <div class="quiz-container">
    <div v-if="currentQuestion">
      <h2>Q{{ currentIndex + 1 }}: {{ currentQuestion.question }}</h2>
      <ul>
        <li v-for="choice in choicesList" :key="choice.key">
          <button
            :class="{ selected: selectedKeys.includes(choice.key) }"
            :disabled="answered"
            @click="toggleSelect(choice.key)"
            type="button"
          >
            {{ choice.key }}. {{ choice.text }}
          </button>
        </li>
      </ul>
      <div v-if="!answered">
        <button :disabled="selectedKeys.length === 0" @click="submitAnswer">回答する</button>
        <div class="nav-buttons">
          <button @click="prevQuestion" :disabled="currentIndex === 0">戻る</button>
          <button @click="goNextQuestion" :disabled="currentIndex === questions.length - 1">次へ</button>
        </div>
      </div>
      <div v-else>
        <p :class="{ correct: isCorrect, incorrect: !isCorrect }">
          {{ isCorrect ? '正解！' : '不正解' }}（正解: {{ answerDisplay }}）
        </p>
        <p class="explanation" v-html="explanationWithBr"></p>
        <button @click="nextQuestion">次の問題へ</button>
      </div>
    </div>
    <div v-else>
      <h2>全ての問題が終了しました</h2>
      <div class="result">
        <p>正答数: {{ correctCount }}/{{ questions.length }}　正答率: {{ correctRate }}%</p>
      </div>
      <button @click="restartQuiz">最初からやり直す</button>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, computed } from 'vue';
import Papa from 'papaparse';

interface Question {
  question: string;
  choices: string; // "A.選択肢1;B.選択肢2;..." 形式
  answer: string; // 例: "B" または "AE"
  explanation: string;
}

type ChoiceKey = string;

export default defineComponent({
  name: 'QuestionQuiz',
  setup() {
    const questions = ref<Question[]>([]);
    const currentIndex = ref(0);
    const answered = ref(false);
    const isCorrect = ref(false);
    const selectedKeys = ref<ChoiceKey[]>([]);
    const choicesList = ref<{ key: ChoiceKey; text: string }[]>([]);
    // 正答数カウント
    const correctCount = ref(0);

    // CSV読み込み
    const loadQuestions = () => {
      fetch(import.meta.env.BASE_URL + 'questions.csv')
        .then((res) => res.text())
        .then((csv) => {
          const parsed = Papa.parse<Question>(csv, { header: true });
          // 10問ランダム抽出
          const all = shuffleArray(parsed.data.filter(q => q.question));
          questions.value = all.slice(0, 10);
          setChoicesList();
        });
    };

    // 選択肢リストをセット（ランダムシャッフル＋表示順でアルファベット付与）
    const setChoicesList = () => {
      const q = questions.value[currentIndex.value];
      if (!q) return;
      const parsedChoices = q.choices.split(';').map((c) => {
        const m = c.match(/^[A-Z]\.(.*)$/);
        return m ? m[1] : c;
      }).filter(Boolean);
      // シャッフル
      const shuffled = shuffleArray(parsedChoices);
      // 表示順でA,B,C...を付与
      choicesList.value = shuffled.map((text, idx) => ({
        key: String.fromCharCode(65 + idx),
        text
      }));
      selectedKeys.value = [];
    };

    // 配列シャッフル
    function shuffleArray<T>(array: T[]): T[] {
      return array
        .map(value => ({ value, sort: Math.random() }))
        .sort((a, b) => a.sort - b.sort)
        .map(({ value }) => value);
    }

    // 選択肢の選択/解除
    const toggleSelect = (key: ChoiceKey) => {
      if (answered.value) return;
      if (selectedKeys.value.includes(key)) {
        selectedKeys.value = selectedKeys.value.filter(k => k !== key);
      } else {
        selectedKeys.value.push(key);
      }
    };

    // 回答確定
    const submitAnswer = () => {
      answered.value = true;
      const q = questions.value[currentIndex.value];
      // 正解キーをchoicesの元の順序から再マッピング
      const originalChoices = q.choices.split(';').map((c) => {
        const m = c.match(/^([A-Z])\.(.*)$/);
        return m ? { key: m[1], text: m[2] } : null;
      }).filter(Boolean) as { key: string; text: string }[];
      // シャッフル後の表示順で正解キーを再構成
      const answerKeys = q.answer.split('').map(origKey => {
        const orig = originalChoices.find(c => c.key === origKey);
        if (!orig) return '';
        const idx = choicesList.value.findIndex(c => c.text === orig.text);
        return idx >= 0 ? choicesList.value[idx].key : '';
      }).filter(Boolean).sort();
      const selectedSorted = [...selectedKeys.value].sort();
      isCorrect.value = JSON.stringify(selectedSorted) === JSON.stringify(answerKeys);
      if (isCorrect.value) correctCount.value++;
    };

    // 正解の表示用（表示順アルファベットで）
    const answerDisplay = computed(() => {
      const q = questions.value[currentIndex.value];
      if (!q) return '';
      // 正解キーをchoicesの元の順序から再マッピング
      const originalChoices = q.choices.split(';').map((c) => {
        const m = c.match(/^([A-Z])\.(.*)$/);
        return m ? { key: m[1], text: m[2] } : null;
      }).filter(Boolean) as { key: string; text: string }[];
      // シャッフル後の表示順で正解キーを再構成
      const answerKeys = q.answer.split('').map(origKey => {
        const orig = originalChoices.find(c => c.key === origKey);
        if (!orig) return '';
        const idx = choicesList.value.findIndex(c => c.text === orig.text);
        return idx >= 0 ? choicesList.value[idx].key : '';
      }).filter(Boolean).sort();
      return answerKeys.join('');
    });

    // 解説の改行を<br>に変換
    const explanationWithBr = computed(() => {
      if (!currentQuestion.value) return '';
      return currentQuestion.value.explanation.replace(/\n/g, '<br>');
    });

    // 次の問題
    const nextQuestion = () => {
      currentIndex.value++;
      answered.value = false;
      selectedKeys.value = [];
      if (currentIndex.value < questions.value.length) {
        setChoicesList();
      }
    };

    // 最初から
    const restartQuiz = () => {
      currentIndex.value = 0;
      answered.value = false;
      selectedKeys.value = [];
      correctCount.value = 0;
      questions.value = shuffleArray(questions.value);
      setChoicesList();
    };

    // 現在の問題
    const currentQuestion = computed(() => questions.value[currentIndex.value]);

    // --- 追加: 戻る・次へ機能 ---
    const prevQuestion = () => {
      if (currentIndex.value > 0) {
        currentIndex.value--;
        answered.value = false;
        selectedKeys.value = [];
        setChoicesList();
      }
    };
    const goNextQuestion = () => {
      if (currentIndex.value < questions.value.length - 1) {
        currentIndex.value++;
        answered.value = false;
        selectedKeys.value = [];
        setChoicesList();
      }
    };

    // 正答率
    const correctRate = computed(() => {
      if (questions.value.length === 0) return 0;
      return Math.round((correctCount.value / questions.value.length) * 100);
    });

    // 初期化
    loadQuestions();

    return {
      currentQuestion,
      currentIndex,
      answered,
      isCorrect,
      choicesList,
      selectedKeys,
      toggleSelect,
      submitAnswer,
      nextQuestion,
      restartQuiz,
      answerDisplay,
      explanationWithBr,
      prevQuestion,
      goNextQuestion,
      questions,
      correctCount,
      correctRate,
    };
  },
});
</script>

<style scoped>
.quiz-container {
  max-width: 600px;
  margin: 2rem auto;
  padding: 2rem;
  background: #fff;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.08);
}
ul {
  list-style: none;
  padding: 0;
}
button {
  margin: 0.5rem 0;
  padding: 0.5rem 1rem;
  font-size: 1rem;
}
button.selected {
  background: #42b883;
  color: #fff;
}
.correct {
  color: green;
}
.incorrect {
  color: red;
}
.explanation {
  margin-top: 1rem;
  background: #f8f8f8;
  padding: 1rem;
  border-radius: 4px;
}
.nav-buttons {
  display: flex;
  justify-content: space-between;
  margin-top: 1rem;
}
.result {
  margin-top: 1rem;
  font-size: 1.2rem;
}
</style>
