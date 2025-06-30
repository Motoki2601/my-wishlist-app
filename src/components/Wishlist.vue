
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
      sortableItems.sort((a, b) => a.id - b.id);
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
  max-width: 850px;
  margin: 0 auto;
  padding: 1.5rem; /* rem単位に変更 */
  background-color: #ffffff;
  border-radius: 0.8rem; /* rem単位に変更 */
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.08);
  border: 1px solid #e0e0e0;
}

h2 {
  color: #6a9955;
  margin-bottom: 1.8rem; /* rem単位に変更 */
  font-weight: 600;
  font-size: 1.9rem; /* rem単位に変更 */
}

ul {
  list-style: none;
  padding: 0;
}

.wishlist-item {
  background-color: #fcfcfc;
  border: 1px solid #ececec;
  border-radius: 0.6rem; /* rem単位に変更 */
  margin-bottom: 0.8rem; /* rem単位に変更 */
  padding: 1.2rem 1.4rem; /* rem単位に変更 */
  text-align: left;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  transition: transform 0.2s ease, box-shadow 0.2s ease;
}

.wishlist-item:hover {
  transform: translateY(-3px);
  box-shadow: 0 6px 15px rgba(0, 0, 0, 0.1);
}

.item-info {
  flex-grow: 1;
  padding-right: 1.2rem; /* rem単位に変更 */
}

.item-info h3 {
  color: #4a4a4a;
  margin-top: 0;
  margin-bottom: 0.5rem; /* rem単位に変更 */
  font-size: 1.4rem; /* rem単位に変更 */
}

.item-info p {
  margin: 0.25rem 0; /* rem単位に変更 */
  color: #666;
  font-size: 0.95rem; /* rem単位に変更 */
}

.item-info a {
  color: #72a85e;
  text-decoration: none;
  font-weight: 500;
  display: inline-block;
  margin-top: 0.5rem; /* rem単位に変更 */
  padding: 0.25rem 0; /* rem単位に変更 */
  border-bottom: 1px dashed #72a85e;
}

.item-info a:hover {
  color: #5b8748;
  border-bottom-color: #5b8748;
}

.item-memo {
  font-size: 0.88rem; /* rem単位に変更 */
  color: #888;
  margin-top: 0.6rem; /* rem単位に変更 */
  line-height: 1.5;
}

small {
  display: block;
  margin-top: 0.75rem; /* rem単位に変更 */
  font-size: 0.78rem; /* rem単位に変更 */
  color: #a0a0a0;
}

.no-items-message {
  text-align: center;
  color: #888;
  margin-top: 2rem; /* rem単位に変更 */
  padding: 1.2rem; /* rem単位に変更 */
  background-color: #f2f2f2;
  border-radius: 0.5rem; /* rem単位に変更 */
  border: 1px dashed #cccccc;
  font-size: 1rem; /* rem単位に変更 */
}

.controls {
  text-align: right;
  margin-bottom: 1.5rem; /* rem単位に変更 */
  display: flex;
  gap: 1.2rem; /* rem単位に変更 */
  justify-content: flex-end;
  flex-wrap: wrap; /* 折り返しを有効にする */
  padding-bottom: 1rem; /* rem単位に変更 */
  border-bottom: 1px solid #eee;
}

.control-group {
  display: flex;
  align-items: center;
  background-color: #f7f7f7;
  padding: 0.6rem 0.8rem; /* rem単位に変更 */
  border-radius: 0.4rem; /* rem単位に変更 */
  border: 1px solid #e5e5e5;
}

.controls label {
  margin-right: 0.6rem; /* rem単位に変更 */
  font-weight: 500;
  white-space: nowrap;
  color: #555;
  font-size: 0.92rem; /* rem単位に変更 */
}

.controls select,
.controls input[type="checkbox"] {
  padding: 0.5rem; /* rem単位に変更 */
  border: 1px solid #dcdcdc;
  border-radius: 0.3rem; /* rem単位に変更 */
  font-size: 0.92rem; /* rem単位に変更 */
  color: #4a4a4a;
}

.controls select {
  background-color: #ffffff;
  cursor: pointer;
}

.item-actions {
  display: flex;
  flex-direction: column;
  gap: 0.6rem; /* rem単位に変更 */
  margin-left: 1.2rem; /* rem単位に変更 */
}

.purchase-button, .edit-button, .delete-button {
  border: none;
  padding: 0.6rem 1rem; /* rem単位に変更 */
  border-radius: 1.2rem; /* rem単位に変更 */
  cursor: pointer;
  transition: background-color 0.3s ease, transform 0.2s ease, box-shadow 0.2s ease;
  font-size: 0.9rem; /* rem単位に変更 */
  white-space: nowrap;
  font-weight: 500;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.08);
  width: 100%; /* ボタンの幅を調整 */
}

.purchase-button {
  background-color: #72a85e;
  color: white;
}

.purchase-button:hover {
  background-color: #5b8748;
  transform: translateY(-1px);
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.12);
}

.edit-button {
  background-color: #f0f0f0;
  color: #555;
  border: 1px solid #ddd;
}

.edit-button:hover {
  background-color: #e0e0e0;
  transform: translateY(-1px);
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.12);
}

.delete-button {
  background-color: #e57373;
  color: white;
}

.delete-button:hover {
  background-color: #d35a5a;
  transform: translateY(-1px);
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.12);
}

.purchased-label {
  color: #6a9955;
  font-weight: bold;
  font-size: 0.95rem; /* rem単位に変更 */
  white-space: nowrap;
  padding: 0.5rem 0.6rem; /* rem単位に変更 */
  background-color: #e6ffe6;
  border-radius: 1.2rem; /* rem単位に変更 */
  border: 1px solid #a4d8a4;
  text-align: center;
}

.wishlist-item.is-purchased {
  opacity: 0.8;
  text-decoration: line-through;
  background-color: #f0fff0;
  box-shadow: 0 1px 4px rgba(0, 0, 0, 0.03);
}

/* --- メディアクエリ（スマートフォン向け） --- */
@media (max-width: 768px) {
  .wishlist-container {
    padding: 1rem; /* スマホではパディングを小さく */
    border-radius: 0.5rem; /* スマホでは角丸を少し小さく */
  }

  h2 {
    font-size: 1.6rem; /* スマホではタイトルを小さく */
    margin-bottom: 1.2rem;
  }

  .controls {
    flex-direction: column; /* コントロールを縦並びに */
    align-items: flex-start; /* 左揃えに */
    gap: 0.8rem; /* 間隔を調整 */
    padding-bottom: 0.8rem;
  }

  .control-group {
    width: 100%; /* コントロールグループの幅を100%に */
    justify-content: space-between; /* 要素を両端に寄せる */
    padding: 0.5rem 0.6rem;
  }

  .controls label,
  .controls select,
  .controls input[type="checkbox"] {
    font-size: 0.9rem; /* スマホではコントロールのフォントを小さく */
  }
  
  .wishlist-item {
    flex-direction: column; /* アイテムを縦並びに変更 */
    align-items: stretch; /* 幅いっぱいに広げる */
    padding: 1rem; /* スマホではアイテムのパディングを小さく */
  }

  .item-info {
    padding-right: 0; /* 右パディングをリセット */
    margin-bottom: 1rem; /* 情報とボタンの間に余白 */
  }

  .item-info h3 {
    font-size: 1.2rem; /* スマホでは商品名を小さく */
  }

  .item-info p,
  .item-info a,
  .item-memo,
  small {
    font-size: 0.85rem; /* スマホではテキストを小さく */
  }

  .item-actions {
    flex-direction: row; /* ボタンを横並びに変更 */
    flex-wrap: wrap; /* ボタンが収まらない場合に折り返す */
    justify-content: center; /* 中央寄せ */
    margin-left: 0; /* マージンをリセット */
    gap: 0.5rem; /* ボタン間の間隔を調整 */
  }

  .purchase-button, .edit-button, .delete-button {
    flex: 1 1 auto; /* ボタンが自動的に伸縮する */
    min-width: 45%; /* 最小幅を設定して2列になるように調整 */
    max-width: 50%;
    font-size: 0.85rem; /* スマホではボタンのフォントサイズを小さく */
    padding: 0.7rem 0.8rem;
  }

  .purchased-label {
    font-size: 0.85rem;
    padding: 0.6rem 0.8rem;
    width: 100%; /* 購入済みラベルも幅いっぱいに */
    box-sizing: border-box;
  }
}
</style>