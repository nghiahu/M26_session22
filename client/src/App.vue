<template>
  <div>
    List product
    <form @submit.prevent="handleSubmit">
      <div>
        <label for="">Name</label>
        <input v-model="productInfo.name" type="text" />
      </div>
      <div>
        <label for="">Price</label>
        <input v-model="productInfo.price" type="text" />
      </div>
      <div>
        <label for="">Quantity</label>
        <input v-model="productInfo.quantity" type="text" />
      </div>
      <button>{{ typeForm === 'add' ? 'Add' : 'Update' }}</button>
    </form>
    <table border="1">
      <thead>
        <tr>
          <td>id</td>
          <td>name</td>
          <td>price</td>
          <td>quantity</td>
          <td>Option</td>
        </tr>
      </thead>
      <tbody>
        <tr v-for="item in products" :key="item.id">
          <td>{{ item.id }}</td>
          <td>{{ item.name }}</td>
          <td>{{ item.price }}</td>
          <td>{{ item.quantity }}</td>
          <td>
            <button @click="handleEdit(item.id)">Sửa</button>
            <button @click="handleDelete(item.id)">Xóa</button>
          </td>
        </tr>
      </tbody>
    </table>
    <div v-if="isShowLoading">Loading...</div>
    <Bai789></Bai789>
  </div>
</template>

<script setup>
import Bai789 from './components/baitap/bai789.vue'
import { onMounted, reactive, ref } from "vue";
const isShowLoading = ref(false);
const products = ref([]);
const baseId = ref(null);
const loadData = async () => {
  try {
    isShowLoading.value = true;

    const response = await fetch("http://localhost:8080/products");
    const data = await response.json();
    products.value = data;
  } catch (errer) {
    return errer;
  } finally {
    isShowLoading.value = false;
  }
};

onMounted(() => {
  loadData();
});

const typeForm = ref("add");
const productInfo = reactive({
  name: "",
  price: 0,
  quantity: 0,
});
const currentEditId = ref(null);  // Lưu id của sản phẩm đang sửa

const handleSubmit = async () => {
  if (typeForm.value === "add") {
    const response = await fetch("http://localhost:8080/products", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify(productInfo),
    });
    console.log(response);
  } else if (typeForm.value === "edit") {
    // Thực hiện PUT để cập nhật sản phẩm
    const response = await fetch(`http://localhost:8080/products/${currentEditId.value}`, {
      method: "PUT",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify(productInfo),
    });
    console.log(response);
    typeForm.value = "add";  // Sau khi cập nhật, chuyển lại thành form 'add'
  }
  loadData();
  resetForm();  // Xóa sạch form sau khi thêm hoặc cập nhật
};

const handleDelete = async (id) => {
  const response = await fetch(`http://localhost:8080/products/${id}`, {
    method: "DELETE",
  });
  if (response.status === 200) {
    loadData();
  }
};

const handleEdit = async (id) => {
  const response = await fetch(`http://localhost:8080/products/${id}`, {
    method: "GET",
  });
  const data = await response.json();
  if (data) {
    productInfo.name = data.name;
    productInfo.price = data.price;
    productInfo.quantity = data.quantity;
    currentEditId.value = id;  // Lưu id sản phẩm cần sửa
    typeForm.value = "edit";   // Chuyển form sang chế độ sửa
  }
};

const resetForm = () => {
  productInfo.name = "";
  productInfo.price = 0;
  productInfo.quantity = 0;
  currentEditId.value = null;  // Xóa id khi reset form
};
</script>

<style></style>