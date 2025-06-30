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
  background-color: rgba(0, 0, 0, 0.4);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
  backdrop-filter: blur(3px);
}

.modal-container {
  background-color: #fcfcfc;
  padding: 2.2rem 2.5rem; /* rem単位に変更 */
  border-radius: 0.8rem; /* rem単位に変更 */
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.15);
  max-width: 650px;
  width: 90%; /* パーセントで指定 */
  position: relative;
  box-sizing: border-box;
}

.close-button {
  position: absolute;
  top: 1rem; /* rem単位に変更 */
  right: 1rem; /* rem単位に変更 */
  background: none;
  border: none;
  font-size: 1.8rem; /* rem単位に変更 */
  cursor: pointer;
  color: #888;
  padding: 0.3rem; /* rem単位に変更 */
  line-height: 1;
  transition: color 0.3s ease;
}

.close-button:hover {
  color: #555;
}

h2 {
  color: #6a9955;
  text-align: center;
  margin-bottom: 2rem; /* rem単位に変更 */
  margin-top: 0;
  font-weight: 600;
  font-size: 1.8rem; /* rem単位に変更 */
}

.form-group {
  margin-bottom: 1.2rem; /* rem単位に変更 */
}

label {
  display: block;
  margin-bottom: 0.5rem; /* rem単位に変更 */
  font-weight: 500;
  color: #555;
  font-size: 0.95rem; /* rem単位に変更 */
}

input[type="text"],
input[type="number"],
input[type="url"],
textarea,
select {
  width: 100%;
  padding: 0.75rem; /* rem単位に変更 */
  border: 1px solid #dcdcdc;
  border-radius: 0.4rem; /* rem単位に変更 */
  font-size: 1rem; /* rem単位に変更 */
  box-sizing: border-box;
  transition: border-color 0.3s ease, box-shadow 0.3s ease;
}

input[type="text"]:focus,
input[type="number"]:focus,
input[type="url"]:focus,
textarea:focus,
select:focus {
  outline: none;
  border-color: #8bbd77;
  box-shadow: 0 0 0 3px rgba(106, 153, 85, 0.2);
}

textarea {
  resize: vertical;
  min-height: 6rem; /* rem単位に変更 */
}

button[type="submit"] {
  background-color: #6a9955;
  color: white;
  padding: 0.8rem 1.5rem; /* rem単位に変更 */
  border-radius: 1.5rem; /* rem単位に変更 */
  font-size: 1.1rem; /* rem単位に変更 */
  margin-top: 1.5rem; /* rem単位に変更 */
  box-shadow: 0 3px 8px rgba(0, 0, 0, 0.1);
  width: 100%; /* 幅を100%に設定 */
}

button[type="submit"]:hover {
  background-color: #5b8748;
  transform: translateY(-1px);
}

.cancel-button {
  background-color: #a8a8a8;
  color: white;
  margin-top: 0.8rem; /* rem単位に変更 */
  padding: 0.8rem 1.5rem; /* rem単位に変更 */
  border-radius: 1.5rem; /* rem単位に変更 */
  font-size: 1.1rem; /* rem単位に変更 */
  box-shadow: 0 3px 8px rgba(0, 0, 0, 0.1);
  width: 100%; /* 幅を100%に設定 */
}

.cancel-button:hover {
  background-color: #909090;
  transform: translateY(-1px);
}

/* --- メディアクエリ（スマートフォン向け） --- */
@media (max-width: 768px) {
  .modal-container {
    padding: 1.5rem 1.2rem; /* スマホではパディングを小さく */
    width: 95%; /* スマホでは幅を広げる */
  }

  h2 {
    font-size: 1.5rem; /* スマホではタイトルを小さく */
    margin-bottom: 1.5rem;
  }

  label {
    font-size: 0.9rem;
  }

  input[type="text"],
  input[type="number"],
  input[type="url"],
  textarea,
  select {
    padding: 0.6rem; /* スマホでは入力欄のパディングを小さく */
    font-size: 0.9rem;
  }

  textarea {
    min-height: 5rem; /* スマホではテキストエリアの最小高さを小さく */
  }

  button[type="submit"],
  .cancel-button {
    font-size: 1rem; /* スマホではボタンのフォントサイズを小さく */
    padding: 0.7rem 1rem;
  }
}
</style>