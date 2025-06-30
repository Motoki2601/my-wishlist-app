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
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #4a4a4a;
  margin-top: 0; /* body側のマージンに任せる */
  background-color: #f7f7f7;
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
}

header {
  margin-bottom: 2rem; /* rem単位に変更 */
  width: 100%;
  padding: 1.5rem 0; /* rem単位に変更 */
  background-color: #e0e0e0;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
}

h1 {
  color: #6a9955;
  font-weight: 600;
  font-size: 2.2rem; /* rem単位に変更 */
}

main {
  width: 95%; /* 幅をパーセントで指定 */
  max-width: 850px;
  padding: 0 1rem; /* rem単位に変更 */
  box-sizing: border-box;
}

.add-new-item-button {
  background-color: #6a9955;
  color: white;
  border: none;
  padding: 1rem 2rem; /* rem単位に変更 */
  border-radius: 2rem; /* rem単位に変更 */
  font-size: 1.15rem; /* rem単位に変更 */
  cursor: pointer;
  transition: background-color 0.3s ease, transform 0.2s ease;
  margin-top: 1.5rem; /* rem単位に変更 */
  margin-bottom: 1.5rem; /* rem単位に変更 */
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  width: auto; /* 幅を自動調整 */
  max-width: 100%; /* スマホでボタンがはみ出さないように */
}

.add-new-item-button:hover {
  background-color: #5b8748;
  transform: translateY(-2px);
}

/* --- メディアクエリ（スマートフォン向け） --- */
@media (max-width: 768px) {
  h1 {
    font-size: 1.8rem; /* スマホではタイトルを少し小さく */
  }

  main {
    width: 100%; /* スマホではメインコンテンツの幅を100%に */
    padding: 0 0.8rem; /* パディングをさらに小さく */
  }

  .add-new-item-button {
    font-size: 1rem; /* スマホではボタンのフォントサイズを小さく */
    padding: 0.8rem 1.5rem; /* パディングを小さく */
    margin-top: 1rem;
    margin-bottom: 1rem;
  }
}
</style>