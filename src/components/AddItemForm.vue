<template>
  <div class="add-item-form">
    <h2>新しい欲しいものを追加</h2>
    <form @submit.prevent="addItem">
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

      <button type="submit">リストに追加</button>
    </form>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'; // computedもインポートに追加

// イベントを発行するためのemit関数を定義
const emit = defineEmits(['item-added']);

// 新しいアイテムのデータを格納するためのリアクティブな変数
const newItem = ref({
  name: '',
  price: null,
  url: '',
  memo: '',
  category: '',
  priority: '中',
  isPurchased: false
});

// 現在の日付を取得し、YYYY-MM-DD形式にフォーマットするcomputedプロパティ
const currentDate = computed(() => {
  const today = new Date();
  const year = today.getFullYear();
  const month = String(today.getMonth() + 1).padStart(2, '0'); // 月は0から始まるため+1
  const day = String(today.getDate()).padStart(2, '0');
  return `${year}-${month}-${day}`;
});

// アイテム追加処理
const addItem = () => {
  if (!newItem.value.name || newItem.value.price === null) {
    alert('商品名と価格は必須です！');
    return;
  }

  // 新しいアイテムオブジェクトを作成し、登録日を自動設定
  const itemToAdd = {
    ...newItem.value, // 現在のフォームデータをコピー
    id: Date.now(), // 一意のIDを生成 (簡易的な例)
    registrationDate: currentDate.value, // 自動設定された登録日
    isPurchased: false // 初期値は未購入
  };

  // 'item-added'という名前のイベントを発行し、新しいアイテムデータを渡す
  emit('item-added', itemToAdd);

  // フォームをリセット
  newItem.value = {
    name: '',
    price: null,
    url: '',
    memo: '',
    category: '',
    priority: '中',
    isPurchased: false
  };
};
</script>

<style scoped>
/* スタイルは変更なし */
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
  width: calc(100% - 22px); /* paddingとborderを考慮 */
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
}

button:hover {
  background-color: #368a62;
}
</style>