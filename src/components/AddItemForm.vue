<template>
  <div class="add-item-form">
    <h2>{{ isEditing ? '欲しいものを編集' : '新しい欲しいものを追加' }}</h2>
    <form @submit.prevent="submitForm">
      <div class="form-group">
        <label for="name">商品名:</label>
        <input type="text" id="name" v-model="newItem.name" required />
      </div>

      <div class="form-group">
        <label for="price">価格:</label>
        <input type="number" id="price" v-model.number="newItem.price" required />
      </div>

      <div class="form-group">
        <label for="url">WebサイトのURL:</label>
        <input type="url" id="url" v-model="newItem.url" />
      </div>

      <div class="form-group">
        <label for="memo">メモ:</label>
        <textarea id="memo" v-model="newItem.memo"></textarea>
      </div>

      <div class="form-group">
        <label for="category">カテゴリ:</label>
        <input type="text" id="category" v-model="newItem.category" />
      </div>

      <div class="form-group">
        <label for="priority">優先度:</label>
        <select id="priority" v-model="newItem.priority">
          <option value="高">高</option>
          <option value="中">中</option>
          <option value="低">低</option>
        </select>
      </div>

      <button type="submit">{{ isEditing ? '更新' : 'リストに追加' }}</button>
      <button v-if="isEditing" type="button" @click="cancelEdit" class="cancel-button">キャンセル</button>
    </form>
  </div>
</template>

<script setup>
import { ref, computed, watch, defineProps, defineEmits } from 'vue'; // definePropsを追加

// 親から渡されるプロパティを定義
const props = defineProps({
  initialItem: { // 編集中のアイテム情報（新規作成時はnull）
    type: Object,
    default: null
  }
});

// イベントを発行するためのemit関数を定義
const emit = defineEmits(['item-added', 'item-updated', 'cancel-edit']);

// 新しい（または編集中の）アイテムのデータを格納するためのリアクティブな変数
const newItem = ref({
  id: null, // 編集時に必要となるID
  name: '',
  price: null, // 数値として扱うためnull
  url: '',
  memo: '',
  category: '',
  priority: '中', // デフォルト値を設定
  isPurchased: false
});

// フォームが編集モードかどうかを判断する算出プロパティ
const isEditing = computed(() => props.initialItem !== null);

// initialItemプロパティが変更されたときにnewItemを更新
watch(() => props.initialItem, (newInitialItem) => {
  if (newInitialItem) {
    // 編集モードの場合、initialItemの値をnewItemにコピー
    newItem.value = { ...newInitialItem };
  } else {
    // 新規作成モードの場合、newItemをリセット
    resetForm();
  }
}, { immediate: true }); // コンポーネントがマウントされた直後にも実行

// フォームをリセットする関数
const resetForm = () => {
  newItem.value = {
    id: null,
    name: '',
    price: null,
    url: '',
    memo: '',
    category: '',
    priority: '中',
    isPurchased: false
  };
};

// 現在の日付を取得し、YYYY-MM-DD形式にフォーマットする算出プロパティ
const currentDate = computed(() => {
  const today = new Date();
  const year = today.getFullYear();
  const month = String(today.getMonth() + 1).padStart(2, '0');
  const day = String(today.getDate()).padStart(2, '0');
  return `${year}-${month}-${day}`;
});

// フォーム送信処理（新規追加と更新を兼ねる）
const submitForm = () => {
  if (!newItem.value.name || newItem.value.price === null) {
    alert('商品名と価格は必須です！');
    return;
  }

  if (isEditing.value) {
    // 編集モードの場合はitem-updatedイベントを発行
    emit('item-updated', { ...newItem.value });
  } else {
    // 新規作成モードの場合はitem-addedイベントを発行
    const itemToAdd = {
      ...newItem.value,
      id: Date.now(), // 新規作成時のみIDを生成
      registrationDate: currentDate.value
    };
    emit('item-added', itemToAdd);
  }
  resetForm(); // フォームをリセット
};

// 編集キャンセル処理
const cancelEdit = () => {
  emit('cancel-edit'); // 親にキャンセルイベントを通知
  resetForm(); // フォームをリセット
};
</script>

<style scoped>
/* 既存のスタイルは変更なし */
.add-item-form {
  max-width: 600px;
  margin: 30px auto;
  padding: 25px;
  background-color: #ffffff;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.15);
  text-align: left;
}

h2 {
  color: #35495e;
  text-align: center;
  margin-bottom: 25px;
}

.form-group {
  margin-bottom: 15px;
}

label {
  display: block;
  margin-bottom: 5px;
  font-weight: bold;
  color: #333;
}

input[type="text"],
input[type="number"],
input[type="url"],
textarea,
select {
  width: calc(100% - 22px);
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-size: 1em;
}

textarea {
  resize: vertical;
  min-height: 80px;
}

button {
  display: block;
  width: 100%;
  padding: 12px 20px;
  background-color: #42b983;
  color: white;
  border: none;
  border-radius: 4px;
  font-size: 1.1em;
  cursor: pointer;
  transition: background-color 0.3s ease;
  margin-top: 20px; /* ボタンの上に少しスペースを追加 */
}

button[type="submit"] {
  margin-bottom: 10px; /* サブミットボタンの下にスペース */
}

button:hover {
  background-color: #368a62;
}

/* キャンセルボタンのスタイル */
.cancel-button {
  background-color: #6c757d; /* 灰色系 */
  margin-top: 0; /* サブミットボタンとの間隔を調整 */
}

.cancel-button:hover {
  background-color: #5a6268;
}
</style>