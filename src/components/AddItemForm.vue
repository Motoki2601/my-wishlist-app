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

const emit = defineEmits(['item-added', 'item-updated', 'cancel-edit', 'close-modal']);

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

const closeModal = () => {
  emit('close-modal');
  resetForm();
};
</script>

<style scoped>
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.4); /* 半透明の黒を少し薄く */
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
  backdrop-filter: blur(3px); /* 背景を少しぼかす */
}

.modal-container {
  background-color: #fcfcfc; /* モーダルの背景色を明るいグレーに */
  padding: 35px 40px; /* パディングを増やす */
  border-radius: 12px; /* 角をさらに丸く */
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.15); /* 影を強調 */
  max-width: 650px; /* 最大幅を少し広げる */
  width: 90%;
  position: relative;
  box-sizing: border-box;
}

.close-button {
  position: absolute;
  top: 15px;
  right: 15px;
  background: none;
  border: none;
  font-size: 1.8em; /* サイズを少し大きく */
  cursor: pointer;
  color: #888; /* 閉じるボタンの色をグレーに */
  padding: 5px;
  line-height: 1;
  transition: color 0.3s ease;
}

.close-button:hover {
  color: #555;
}

h2 {
  color: #6a9955; /* 緑系の色に */
  text-align: center;
  margin-bottom: 30px; /* 余白を増やす */
  margin-top: 0;
  font-weight: 600;
  font-size: 1.8em;
}

.form-group {
  margin-bottom: 18px; /* 各フォームグループの余白を調整 */
}

label {
  display: block;
  margin-bottom: 7px; /* ラベルと入力欄の間隔を調整 */
  font-weight: 500; /* フォントの太さを調整 */
  color: #555; /* ラベルの色を少し柔らかく */
  font-size: 0.95em;
}

input[type="text"],
input[type="number"],
input[type="url"],
textarea,
select {
  width: 100%;
  padding: 12px; /* パディングを増やす */
  border: 1px solid #dcdcdc; /* ボーダーの色を薄いグレーに */
  border-radius: 6px; /* 角を丸く */
  font-size: 1em;
  box-sizing: border-box; /* パディングをwidthに含める */
  transition: border-color 0.3s ease, box-shadow 0.3s ease;
}

input[type="text"]:focus,
input[type="number"]:focus,
input[type="url"]:focus,
textarea:focus,
select:focus {
  outline: none;
  border-color: #8bbd77; /* フォーカス時のボーダー色を緑系に */
  box-shadow: 0 0 0 3px rgba(106, 153, 85, 0.2); /* フォーカス時の影を追加 */
}

textarea {
  resize: vertical;
  min-height: 100px; /* 最小高さを増やす */
}

button[type="submit"] {
  background-color: #6a9955; /* 緑系の色に */
  color: white;
  padding: 14px 25px;
  border-radius: 25px; /* 角を丸く */
  font-size: 1.1em;
  margin-top: 25px; /* 上部の余白を調整 */
  box-shadow: 0 3px 8px rgba(0, 0, 0, 0.1);
}

button[type="submit"]:hover {
  background-color: #5b8748;
  transform: translateY(-1px);
}

.cancel-button {
  background-color: #a8a8a8; /* グレー系の色に */
  color: white;
  margin-top: 10px; /* ボタン間の余白 */
  padding: 14px 25px;
  border-radius: 25px; /* 角を丸く */
  font-size: 1.1em;
  box-shadow: 0 3px 8px rgba(0, 0, 0, 0.1);
}

.cancel-button:hover {
  background-color: #909090;
  transform: translateY(-1px);
}
</style>