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
          
          <button @click="editItem(item.id)" class="edit-button">編集</button> 
          <button @click="deleteItem(item.id)" class="delete-button">削除</button>
        </div>
      </li>
    </ul>
  </div>
</template>

<script setup>
import { defineProps, ref, computed, defineEmits } from 'vue';

const props = defineProps({
  items: {
    type: Array,
    required: true
  }
});

const emit = defineEmits(['update-item-status', 'delete-item', 'edit-item']);

const currentSort = ref('none');
const selectedCategory = ref('');
const showUnpurchasedOnly = ref(false);

const uniqueCategories = computed(() => {
  const categories = new Set();
  props.items.forEach(item => {
    if (item.category) {
      categories.add(item.category);
    }
  });
  return [...categories].sort();
});

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
      sortableItems.sort((a, b) => a.id - b.id); // デフォルトはID順（追加順）
      break;
  }
  return sortableItems;
});

const filteredAndSortedItems = computed(() => {
  let filteredItems = sortedItems.value; 

  if (selectedCategory.value) {
    filteredItems = filteredItems.filter(item => item.category === selectedCategory.value);
  }

  if (showUnpurchasedOnly.value) {
    filteredItems = filteredItems.filter(item => !item.isPurchased);
  }

  return filteredItems;
});

const markAsPurchased = (itemId) => {
  emit('update-item-status', itemId, true);
};

const deleteItem = (itemId) => {
  if (confirm('この欲しいものを削除してもよろしいですか？')) {
    emit('delete-item', itemId);
  }
};

const editItem = (itemId) => {
  emit('edit-item', itemId);
};
</script>

<style scoped>
.wishlist-container {
  max-width: 850px; /* App.vue と合わせて最大幅を調整 */
  margin: 0 auto;
  padding: 25px; /* パディングを調整 */
  background-color: #ffffff; /* 白い背景に */
  border-radius: 10px; /* 角を丸く */
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.08); /* 影を強調 */
  border: 1px solid #e0e0e0; /* 薄いグレーのボーダーを追加 */
}

h2 {
  color: #6a9955; /* 緑系の色に */
  margin-bottom: 25px; /* 余白を調整 */
  font-weight: 600;
  font-size: 1.9em;
}

ul {
  list-style: none;
  padding: 0;
}

.wishlist-item {
  background-color: #fcfcfc; /* アイテムの背景色をより明るく */
  border: 1px solid #ececec; /* ボーダーを薄く、柔らかい色に */
  border-radius: 8px; /* 角を丸く */
  margin-bottom: 12px; /* アイテム間の余白 */
  padding: 18px 20px; /* パディングを調整 */
  text-align: left;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05); /* 影を柔らかく */
  display: flex;
  justify-content: space-between;
  align-items: flex-start; /* 上揃えに */
  transition: transform 0.2s ease, box-shadow 0.2s ease;
}

.wishlist-item:hover {
  transform: translateY(-3px); /* ホバー時に少し浮き上がる */
  box-shadow: 0 6px 15px rgba(0, 0, 0, 0.1);
}

.item-info {
  flex-grow: 1;
  padding-right: 20px; /* 情報とボタンの間にスペース */
}

.item-info h3 {
  color: #4a4a4a; /* 商品名の色を濃いグレーに */
  margin-top: 0;
  margin-bottom: 8px; /* 下部余白を調整 */
  font-size: 1.4em;
}

.item-info p {
  margin: 4px 0; /* 行間の余白を調整 */
  color: #666; /* 文字色を柔らかいグレーに */
  font-size: 0.95em;
}

.item-info a {
  color: #72a85e; /* 緑系のリンク色 */
  text-decoration: none;
  font-weight: 500;
  display: inline-block; /* リンクを下線なしで表示し、少しボタンのように */
  margin-top: 8px;
  padding: 4px 0;
  border-bottom: 1px dashed #72a85e; /* 破線の下線 */
}

.item-info a:hover {
  color: #5b8748;
  border-bottom-color: #5b8748;
}

.item-memo {
  font-size: 0.88em;
  color: #888;
  margin-top: 10px;
  line-height: 1.5; /* 行の高さを調整 */
}

small {
  display: block;
  margin-top: 12px;
  font-size: 0.78em;
  color: #a0a0a0;
}

.no-items-message {
  text-align: center;
  color: #888;
  margin-top: 40px;
  padding: 20px;
  background-color: #f2f2f2;
  border-radius: 8px;
  border: 1px dashed #cccccc;
}

.controls {
  text-align: right;
  margin-bottom: 25px; /* コントロールとリストの間の余白 */
  display: flex;
  gap: 20px; /* 各コントロールグループの間隔 */
  justify-content: flex-end;
  flex-wrap: wrap; /* 小さい画面での折り返し */
  padding-bottom: 15px; /* コントロールの下部にパディング */
  border-bottom: 1px solid #eee; /* 下線を追加 */
}

.control-group {
  display: flex;
  align-items: center;
  background-color: #f7f7f7; /* コントロールグループの背景色 */
  padding: 8px 12px;
  border-radius: 6px;
  border: 1px solid #e5e5e5;
}

.controls label {
  margin-right: 10px;
  font-weight: 500;
  white-space: nowrap;
  color: #555;
  font-size: 0.92em;
}

.controls select,
.controls input[type="checkbox"] {
  padding: 8px;
  border: 1px solid #dcdcdc;
  border-radius: 5px; /* 角を少し丸く */
  font-size: 0.92em;
  color: #4a4a4a;
}

.controls select {
  background-color: #ffffff;
  cursor: pointer;
}

/* アクションボタン用のスタイル */
.item-actions {
  display: flex;
  flex-direction: column;
  gap: 8px; /* ボタン間のスペースを少し増やす */
  margin-left: 20px;
}

.purchase-button, .edit-button, .delete-button {
  border: none;
  padding: 10px 15px; /* パディングを増やす */
  border-radius: 20px; /* 角を丸く */
  cursor: pointer;
  transition: background-color 0.3s ease, transform 0.2s ease, box-shadow 0.2s ease;
  font-size: 0.9em;
  white-space: nowrap;
  font-weight: 500;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.08); /* 影を追加 */
}

.purchase-button {
  background-color: #72a85e; /* 緑系の購入ボタン */
  color: white;
}

.purchase-button:hover {
  background-color: #5b8748;
  transform: translateY(-1px);
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.12);
}

.edit-button {
  background-color: #f0f0f0; /* 薄いグレーの編集ボタン */
  color: #555;
  border: 1px solid #ddd;
}

.edit-button:hover {
  background-color: #e0e0e0;
  transform: translateY(-1px);
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.12);
}

.delete-button {
  background-color: #e57373; /* 柔らかい赤系の削除ボタン */
  color: white;
}

.delete-button:hover {
  background-color: #d35a5a;
  transform: translateY(-1px);
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.12);
}

.purchased-label {
  color: #6a9955; /* 緑系の色に */
  font-weight: bold;
  font-size: 0.95em;
  white-space: nowrap;
  padding: 8px 10px;
  background-color: #e6ffe6; /* 薄い緑の背景 */
  border-radius: 20px;
  border: 1px solid #a4d8a4;
  text-align: center;
}

.wishlist-item.is-purchased {
  opacity: 0.8; /* 少し透明感を出す */
  text-decoration: line-through; /* 取り消し線 */
  background-color: #f0fff0; /* 非常に薄い緑の背景 */
  box-shadow: 0 1px 4px rgba(0, 0, 0, 0.03); /* 影をさらに弱く */
}
</style>