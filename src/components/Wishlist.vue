<template>
  <div class="wishlist-container">
    <h2>欲しいものリスト</h2>

    <div class="controls">
      <div class="control-group">
        <label for="sort">ソート:</label>
        <select id="sort" v-model="currentSort">
          <option value="none">なし</option>
          <option value="nameAsc">商品名 (昇順)</option>
          <option value="nameDesc">商品名 (降順)</option>
          <option value="priceAsc">価格 (安い順)</option>
          <option value="priceDesc">価格 (高い順)</option>
          <option value="dateDesc">登録日 (新しい順)</option>
          <option value="dateAsc">登録日 (古い順)</option>
          <option value="priorityHigh">優先度 (高→低)</option>
        </select>
      </div>

      <div class="control-group">
        <label for="categoryFilter">カテゴリ:</label>
        <select id="categoryFilter" v-model="selectedCategory">
          <option value="">すべてのカテゴリ</option>
          <option v-for="category in uniqueCategories" :key="category" :value="category">{{ category }}</option>
        </select>
      </div>

      <div class="control-group">
        <label for="showOnlyUnpurchased">未購入のみ表示:</label>
        <input type="checkbox" id="showOnlyUnpurchased" v-model="showUnpurchasedOnly">
      </div>
    </div>

    <p v-if="filteredAndSortedItems.length === 0" class="no-items-message">
      {{ items.length === 0 ? 'まだ欲しいものがありません。追加してみましょう！' : '条件に合う欲しいものが見つかりませんでした。' }}
    </p>
    
    <ul>
      <li v-for="item in filteredAndSortedItems" :key="item.id" class="wishlist-item" :class="{ 'is-purchased': item.isPurchased }">
        <div class="item-info">
          <h3>{{ item.name }}</h3>
          <p>価格: {{ item.price }}円</p>
          <p>カテゴリ: {{ item.category }}</p>
          <p>優先度: {{ item.priority }}</p>
          <a :href="item.url" target="_blank" rel="noopener noreferrer">商品ページを見る</a>
          <p class="item-memo">{{ item.memo }}</p>
          <small>登録日: {{ item.registrationDate }}</small>
        </div>
        <div class="item-actions">
          <button v-if="!item.isPurchased" @click="markAsPurchased(item.id)" class="purchase-button">購入済みにする</button>
          <span v-else class="purchased-label">購入済み</span>
        </div>
      </li>
    </ul>
  </div>
</template>

<script setup>
import { defineProps, ref, computed, defineEmits } from 'vue'; // defineEmitsをインポートに追加

const props = defineProps({
  items: {
    type: Array,
    required: true
  }
});

const emit = defineEmits(['update-item-status']); // 親にステータス変更を通知するためのイベントを定義

const currentSort = ref('none'); // ソート条件
const selectedCategory = ref(''); // 選択されたカテゴリ
const showUnpurchasedOnly = ref(false); // 未購入のみ表示するかどうか

// 登録されているアイテムからユニークなカテゴリリストを取得
const uniqueCategories = computed(() => {
  const categories = new Set();
  props.items.forEach(item => {
    if (item.category) {
      categories.add(item.category);
    }
  });
  return [...categories].sort();
});

// 欲しいものアイテムをソートするための算出プロパティ (変更なし)
const sortedItems = computed(() => {
  let sortableItems = [...props.items]; 

  switch (currentSort.value) {
    case 'nameAsc':
      sortableItems.sort((a, b) => a.name.localeCompare(b.name));
      break;
    case 'nameDesc':
      sortableItems.sort((a, b) => b.name.localeCompare(a.name));
      break;
    case 'priceAsc':
      sortableItems.sort((a, b) => a.price - b.price);
      break;
    case 'priceDesc':
      sortableItems.sort((a, b) => b.price - a.price);
      break;
    case 'dateDesc':
      sortableItems.sort((a, b) => new Date(b.registrationDate) - new Date(a.registrationDate));
      break;
    case 'dateAsc':
      sortableItems.sort((a, b) => new Date(a.registrationDate) - new Date(b.registrationDate));
      break;
    case 'priorityHigh':
      const priorityOrder = { '高': 3, '中': 2, '低': 1 };
      sortableItems.sort((a, b) => priorityOrder[b.priority] - priorityOrder[a.priority]);
      break;
    case 'none':
    default:
      // ソートなしの場合は元のID順に戻す（データ追加時のIDは増加するため）
      sortableItems.sort((a, b) => a.id - b.id);
      break;
  }
  return sortableItems;
});

// ソートされたアイテムをさらに絞り込むための算出プロパティ
const filteredAndSortedItems = computed(() => {
  let filteredItems = sortedItems.value; 

  // カテゴリでの絞り込み
  if (selectedCategory.value) {
    filteredItems = filteredItems.filter(item => item.category === selectedCategory.value);
  }

  // 未購入のみ表示の絞り込み
  if (showUnpurchasedOnly.value) {
    filteredItems = filteredItems.filter(item => !item.isPurchased);
  }

  return filteredItems;
});

// アイテムを購入済みにマークするメソッド
const markAsPurchased = (itemId) => {
  // 親コンポーネントにアイテムIDと新しいステータス（true）を伝えるイベントを発行
  emit('update-item-status', itemId, true);
};
</script>

<style scoped>
/* 既存のスタイルは変更なし、以下に新規追加スタイル */
.wishlist-container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  background-color: #f9f9f9;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

h2 {
  color: #35495e;
  margin-bottom: 20px;
}

ul {
  list-style: none;
  padding: 0;
}

.wishlist-item {
  background-color: #ffffff;
  border: 1px solid #ddd;
  border-radius: 6px;
  margin-bottom: 15px;
  padding: 15px;
  text-align: left;
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.05);
  display: flex; /* Flexboxを使ってinfoとactionsを横並びにする */
  justify-content: space-between; /* 両端に配置 */
  align-items: center; /* 中央揃え */
}

.item-info {
  flex-grow: 1; /* スペースを埋める */
}

.item-info h3 {
  color: #42b983;
  margin-top: 0;
  margin-bottom: 5px;
}

.item-info p {
  margin: 3px 0;
  color: #555;
}

.item-info a {
  color: #007bff;
  text-decoration: none;
  font-weight: bold;
}

.item-info a:hover {
  text-decoration: underline;
}

.item-memo {
  font-size: 0.9em;
  color: #777;
  margin-top: 8px;
}

small {
  display: block;
  margin-top: 10px;
  font-size: 0.8em;
  color: #999;
}

.no-items-message {
  text-align: center;
  color: #888;
  margin-top: 30px;
}

.controls {
  text-align: right;
  margin-bottom: 20px;
  display: flex;
  gap: 15px;
  justify-content: flex-end;
  flex-wrap: wrap;
}

.control-group {
  display: flex;
  align-items: center;
}

.controls label {
  margin-right: 10px;
  font-weight: bold;
  white-space: nowrap; /* テキストの折り返しを防ぐ */
}

.controls select,
.controls input[type="checkbox"] {
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

/* 新規追加スタイル */
.item-actions {
  margin-left: 20px; /* 情報との間隔 */
}

.purchase-button {
  background-color: #007bff; /* 青系の色 */
  color: white;
  border: none;
  padding: 8px 12px;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.3s ease;
  font-size: 0.9em;
  white-space: nowrap; /* ボタン内のテキストの折り返しを防ぐ */
}

.purchase-button:hover {
  background-color: #0056b3;
}

.purchased-label {
  color: #28a745; /* 緑系の色 */
  font-weight: bold;
  font-size: 0.9em;
  white-space: nowrap;
}

/* 購入済みアイテムのスタイル */
.wishlist-item.is-purchased {
  opacity: 0.7; /* 少し薄く表示 */
  text-decoration: line-through; /* テキストに打ち消し線 */
  background-color: #e6ffe6; /* 少し緑がかった背景色 */
}
</style>