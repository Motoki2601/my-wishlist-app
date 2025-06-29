<template>
  <div id="app">
    <header>
      <h1>私の欲しいものリストアプリ</h1>
    </header>
    <main>
      <Wishlist 
        :items="wishlistItems" 
        @update-item-status="handleUpdateItemStatus" 
        @delete-item="handleDeleteItem"
        @edit-item="handleEditItem"
      />
      
      <AddItemForm 
        :initial-item="editingItem"
        @item-added="handleItemAdded" 
        @item-updated="handleItemUpdated"
        @cancel-edit="cancelEdit"
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

// wishlistItemsが変更されるたびにLocalStorageに保存
watch(wishlistItems, (newItems) => {
  try {
    localStorage.setItem('myWishlistItems', JSON.stringify(newItems));
  } catch (e) {
    console.error("Failed to save items to localStorage", e);
  }
}, { deep: true });

// AddItemFormから新しいアイテムが追加されたときに実行されるメソッド
const handleItemAdded = (newItem) => {
  wishlistItems.value.push(newItem);
  console.log('App.vueで新しいアイテムを受け取りました:', newItem);
};

// Wishlistからアイテムのステータス変更イベントを受け取ったときに実行されるメソッド
const handleUpdateItemStatus = (itemId, newStatus) => {
  const itemIndex = wishlistItems.value.findIndex(item => item.id === itemId);
  if (itemIndex !== -1) {
    wishlistItems.value[itemIndex].isPurchased = newStatus;
    console.log(`アイテムID: ${itemId} の購入ステータスが ${newStatus} に更新されました。`);
  }
};

// Wishlistからアイテム削除イベントを受け取ったときに実行されるメソッド
const handleDeleteItem = (itemId) => {
  wishlistItems.value = wishlistItems.value.filter(item => item.id !== itemId);
  console.log(`アイテムID: ${itemId} が削除されました。`);
  // 削除されたアイテムが編集中のアイテムだった場合、編集モードを終了
  if (editingItem.value && editingItem.value.id === itemId) {
    editingItem.value = null;
  }
};

// Wishlistからアイテム編集イベントを受け取ったときに実行されるメソッド
const handleEditItem = (itemId) => {
  const itemToEdit = wishlistItems.value.find(item => item.id === itemId);
  if (itemToEdit) {
    // 編集中のアイテムとして設定
    editingItem.value = { ...itemToEdit }; // オブジェクトのコピーを渡す
    console.log(`アイテムID: ${itemId} を編集モードにしました。`);
  }
};

// AddItemFormからアイテムが更新されたときに実行されるメソッド
const handleItemUpdated = (updatedItem) => {
  const itemIndex = wishlistItems.value.findIndex(item => item.id === updatedItem.id);
  if (itemIndex !== -1) {
    wishlistItems.value[itemIndex] = updatedItem; // アイテムを更新
    console.log(`アイテムID: ${updatedItem.id} が更新されました。`);
    editingItem.value = null; // 編集モードを終了
  }
};

// AddItemFormから編集キャンセルイベントを受け取ったときに実行されるメソッド
const cancelEdit = () => {
  editingItem.value = null; // 編集モードを終了
  console.log('編集がキャンセルされました。');
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
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

header {
  margin-bottom: 30px;
}

h1 {
  color: #42b983;
}
</style>