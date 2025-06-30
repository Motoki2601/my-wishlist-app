<template>
  <div id="app">
    <header>
      <h1>私の欲しいものリストアプリ</h1>
    </header>
    <main>
      <button @click="openAddFormModal" class="add-new-item-button">＋ 欲しいものを追加</button>

      <Wishlist 
        :items="wishlistItems" 
        @update-item-status="handleUpdateItemStatus" 
        @delete-item="handleDeleteItem"
        @edit-item="handleEditItem"
      />
      
      <AddItemForm 
        v-if="showAddFormModal"
        :initial-item="editingItem"
        @item-added="handleItemAdded" 
        @item-updated="handleItemUpdated"
        @cancel-edit="cancelEdit"
        @close-modal="closeAddFormModal"
      />
    </main>
  </div>
</template>

<script setup>
import { ref, watch, onMounted } from 'vue';
import Wishlist from './components/Wishlist.vue';
import AddItemForm from './components/AddItemForm.vue';

// ローカルストレージからデータを読み込む関数
const loadItemsFromLocalStorage = () => {
  try {
    const storedItems = localStorage.getItem('myWishlistItems');
    return storedItems ? JSON.parse(storedItems) : [];
  } catch (e) {
    console.error("Failed to load items from localStorage", e);
    return [];
  }
};

// 欲しいものアイテムのデータを初期化（LocalStorageから読み込む）
const wishlistItems = ref(loadItemsFromLocalStorage());

// 編集中のアイテムを管理する変数（nullは新規作成モード、オブジェクトは編集モード）
const editingItem = ref(null);

// 欲しいもの追加フォームのポップアップ表示状態
const showAddFormModal = ref(false);

// wishlistItemsが変更されるたびにLocalStorageに保存
watch(wishlistItems, (newItems) => {
  try {
    localStorage.setItem('myWishlistItems', JSON.stringify(newItems));
  } catch (e) {
    console.error("Failed to save items to localStorage", e);
  }
}, { deep: true });

// 新しいアイテムが追加されたときのハンドラ
const handleItemAdded = (newItem) => {
  wishlistItems.value.push(newItem);
  console.log('App.vueで新しいアイテムを受け取りました:', newItem);
  closeAddFormModal();
};

// アイテムのステータス変更ハンドラ
const handleUpdateItemStatus = (itemId, newStatus) => {
  const itemIndex = wishlistItems.value.findIndex(item => item.id === itemId);
  if (itemIndex !== -1) {
    wishlistItems.value[itemIndex].isPurchased = newStatus;
    console.log(`アイテムID: ${itemId} の購入ステータスが ${newStatus} に更新されました。`);
  }
};

// アイテム削除ハンドラ
const handleDeleteItem = (itemId) => {
  wishlistItems.value = wishlistItems.value.filter(item => item.id !== itemId);
  console.log(`アイテムID: ${itemId} が削除されました。`);
  if (editingItem.value && editingItem.value.id === itemId) {
    cancelEdit();
  }
};

// アイテム編集開始ハンドラ
const handleEditItem = (itemId) => {
  const itemToEdit = wishlistItems.value.find(item => item.id === itemId);
  if (itemToEdit) {
    editingItem.value = { ...itemToEdit };
    showAddFormModal.value = true;
    console.log(`アイテムID: ${itemId} を編集モードにしました。`);
  }
};

// アイテム更新ハンドラ
const handleItemUpdated = (updatedItem) => {
  const itemIndex = wishlistItems.value.findIndex(item => item.id === updatedItem.id);
  if (itemIndex !== -1) {
    wishlistItems.value[itemIndex] = updatedItem;
    console.log(`アイテムID: ${updatedItem.id} が更新されました。`);
    closeAddFormModal();
  }
};

// 編集キャンセルハンドラ
const cancelEdit = () => {
  editingItem.value = null;
  closeAddFormModal();
  console.log('編集がキャンセルされました。');
};

// ポップアップを開くメソッド
const openAddFormModal = () => {
  editingItem.value = null;
  showAddFormModal.value = true;
};

// ポップアップを閉じるメソッド
const closeAddFormModal = () => {
  showAddFormModal.value = false;
  editingItem.value = null;
};


// コンポーネントがマウントされた時にローカルストレージにデータがない場合は初期データとして設定（初回起動時のみ）
onMounted(() => {
  if (wishlistItems.value.length === 0 && !localStorage.getItem('myWishlistItems')) {
    const initialItems = [
      {
        id: 1,
        name: 'ワイヤレスイヤホン',
        price: 15000,
        url: 'https://example.com/earphones',
        memo: 'ノイズキャンセリング機能付き。通勤用に欲しい。',
        registrationDate: '2025-06-29',
        category: 'ガジェット',
        priority: '高',
        isPurchased: false
      },
      {
        id: 2,
        name: 'プログラミング入門書',
        price: 3000,
        url: 'https://example.com/programming-book',
        memo: 'Vue.jsの基礎を学ぶため。',
        registrationDate: '2025-06-28',
        category: '書籍',
        priority: '中',
        isPurchased: false
      },
      {
        id: 3,
        name: '新しいリュックサック',
        price: 8500,
        url: 'https://example.com/backpack',
        memo: '容量大きめ、防水機能があると良い。',
        registrationDate: '2025-06-27',
        category: 'ファッション',
        priority: '低',
        isPurchased: false
      }
    ];
    wishlistItems.value = initialItems;
  }
});
</script>

<style scoped>
#app {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; /* フォントを柔らかい印象に */
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #4a4a4a; /* 全体的な文字色をグレー系に */
  margin-top: 40px; /* 上部の余白を調整 */
  background-color: #f7f7f7; /* 全体の背景色を少しグレーに */
  min-height: 100vh; /* 画面全体の高さを確保 */
  display: flex;
  flex-direction: column;
  align-items: center;
}

header {
  margin-bottom: 30px;
  width: 100%; /* 幅を確保 */
  padding: 20px 0;
  background-color: #e0e0e0; /* ヘッダーの背景色をグレーに */
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05); /* 軽い影を追加 */
}

h1 {
  color: #6a9955; /* 緑系の色に */
  font-weight: 600; /* 少し太めに */
  font-size: 2.2em; /* タイトルを少し大きく */
}

main {
  width: 100%;
  max-width: 850px; /* 最大幅を少し広げる */
  padding: 0 20px;
  box-sizing: border-box;
}

/* ポップアップを開くボタンのスタイル */
.add-new-item-button {
  background-color: #6a9955; /* 緑系の色に */
  color: white;
  border: none;
  padding: 14px 30px; /* パディングを少し増やす */
  border-radius: 25px; /* 角を丸くして柔らかさを出す */
  font-size: 1.15em; /* フォントサイズを少し大きく */
  cursor: pointer;
  transition: background-color 0.3s ease, transform 0.2s ease; /* ホバーエフェクトを追加 */
  margin-top: 25px; /* 上部に余白 */
  margin-bottom: 25px; /* 下部に余白 */
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1); /* 影を追加 */
}

.add-new-item-button:hover {
  background-color: #5b8748; /* ホバー時の色を少し濃く */
  transform: translateY(-2px); /* 少し浮き上がるエフェクト */
}
</style>