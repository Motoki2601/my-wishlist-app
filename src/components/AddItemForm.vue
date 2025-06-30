<template>
  <div class="modal-overlay" @click.self="closeModal">
    <div class="modal-container">
      <button class="close-button" @click="closeModal">×</button>
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
  </div>
</template>

<script setup>
import { ref, computed, watch, defineProps, defineEmits } from 'vue';

const props = defineProps({
  initialItem: {
    type: Object,
    default: null
  }
});

const emit = defineEmits(['item-added', 'item-updated', 'cancel-edit', 'close-modal']); // close-modalイベントを追加

const newItem = ref({
  id: null,
  name: '',
  price: null,
  url: '',
  memo: '',
  category: '',
  priority: '中',
  isPurchased: false
});

const isEditing = computed(() => props.initialItem !== null);

watch(() => props.initialItem, (newInitialItem) => {
  if (newInitialItem) {
    newItem.value = { ...newInitialItem };
  } else {
    resetForm();
  }
}, { immediate: true });

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

const currentDate = computed(() => {
  const today = new Date();
  const year = today.getFullYear();
  const month = String(today.getMonth() + 1).padStart(2, '0');
  const day = String(today.getDate()).padStart(2, '0');
  return `${year}-${month}-${day}`;
});

const submitForm = () => {
  if (!newItem.value.name || newItem.value.price === null) {
    alert('商品名と価格は必須です！');
    return;
  }

  if (isEditing.value) {
    emit('item-updated', { ...newItem.value });
  } else {
    const itemToAdd = {
      ...newItem.value,
      id: Date.now(),
      registrationDate: currentDate.value
    };
    emit('item-added', itemToAdd);
  }
};

const cancelEdit = () => {
  emit('cancel-edit');
};

// ★追加: モーダルを閉じるためのメソッド
const closeModal = () => {
  emit('close-modal');
  resetForm(); // フォームもリセット
};
</script>

<style scoped>
/* モーダルオーバーレイのスタイル */
.modal-overlay {
  position: fixed; /* 画面に固定 */
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5); /* 半透明の黒 */
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000; /* 他の要素の上に表示 */
}

/* モーダルコンテナのスタイル */
.modal-container {
  background-color: #ffffff;
  padding: 30px;
  border-radius: 8px;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.25);
  max-width: 600px;
  width: 90%; /* 画面幅に合わせて調整 */
  position: relative; /* 閉じるボタンの配置のため */
  box-sizing: border-box; /* paddingがwidthに含まれるように */
}

/* 閉じるボタンのスタイル */
.close-button {
  position: absolute;
  top: 10px;
  right: 10px;
  background: none;
  border: none;
  font-size: 1.5em;
  cursor: pointer;
  color: #666;
  padding: 5px;
  line-height: 1; /* 余分な高さを削除 */
}

.close-button:hover {
  color: #333;
}

/* フォームとボタンの既存スタイルはそのまま */
.add-item-form {
  max-width: none; /* 親のモーダルコンテナが幅を持つので、ここでは無効化 */
  margin: 0; /* 親のモーダルコンテナが中央寄せするので、ここでは無効化 */
  padding: 0; /* 親のモーダルコンテナがpaddingを持つので、ここでは無効化 */
  background-color: transparent; /* モーダルコンテナが背景色を持つので、ここでは透明に */
  box-shadow: none; /* 親のモーダルコンテナが影を持つので、ここでは無効化 */
}

h2 {
  color: #35495e;
  text-align: center;
  margin-bottom: 25px;
  margin-top: 0; /* 上部パディングを削除 */
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
  width: 100%; /* calcなしで100%に */
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
  margin-top: 20px;
}

button[type="submit"] {
  margin-bottom: 10px;
}

button:hover {
  background-color: #368a62;
}

.cancel-button {
  background-color: #6c757d;
  margin-top: 0;
}

.cancel-button:hover {
  background-color: #5a6268;
}
</style>