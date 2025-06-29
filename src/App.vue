<template>
  <div id="app">
    <header>
      <h1>私の欲しいものリストアプリ</h1>
    </header>
    <main>
      <Wishlist 
        :items="wishlistItems" 
        @update-item-status="handleUpdateItemStatus" 
      />
      
      <AddItemForm @item-added="handleItemAdded" />
    </main>
  </div>
</template>

<script setup>
import { ref, watch, onMounted } from 'vue'; // watchとonMountedをインポートに追加
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

// wishlistItemsが変更されるたびにLocalStorageに保存
watch(wishlistItems, (newItems) => {
  try {
    localStorage.setItem('myWishlistItems', JSON.stringify(newItems));
  } catch (e) {
    console.error("Failed to save items to localStorage", e);
  }
}, { deep: true }); // deep: true で配列内のオブジェクトの変更も監視

// AddItemFormから新しいアイテムが追加されたときに実行されるメソッド
const handleItemAdded = (newItem) => {
  wishlistItems.value.push(newItem);
  console.log('App.vueで新しいアイテムを受け取りました:', newItem);
  console.log('現在のリスト:', wishlistItems.value);
};

// Wishlistからアイテムのステータス変更イベントを受け取ったときに実行されるメソッド
const handleUpdateItemStatus = (itemId, newStatus) => {
  const itemIndex = wishlistItems.value.findIndex(item => item.id === itemId);
  if (itemIndex !== -1) {
    wishlistItems.value[itemIndex].isPurchased = newStatus;
    console.log(`アイテムID: ${itemId} の購入ステータスが ${newStatus} に更新されました。`);
  }
};

// コンポーネントがマウントされた時にダミーデータがない場合は初期データとして設定（初回起動時のみ）
onMounted(() => {
  if (wishlistItems.value.length === 0) {
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
    // localStorageにデータがない場合のみ初期データを設定
    if (!localStorage.getItem('myWishlistItems')) {
      wishlistItems.value = initialItems;
    }
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