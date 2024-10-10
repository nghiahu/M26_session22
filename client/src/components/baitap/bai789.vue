<template>
    <div class="container">
      <div class="left-panel">
        <button @click="toggleForm">Thêm mới sản phẩm</button>
        <table border="1">
          <thead>
            <tr>
              <th>#</th>
              <th>Tên sản phẩm</th>
              <th>Hình ảnh</th>
              <th>Giá</th>
              <th>Số lượng (kg)</th>
              <th>Ngày thêm</th>
              <th>Chức năng</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(item, index) in products" :key="item.id">
              <td>{{ index + 1 }}</td>
              <td>{{ item.name }}</td>
              <td>
                <img :src="item.imageUrl" alt="Product Image" width="50" />
              </td>
              <td>{{ item.price }} đ</td>
              <td>{{ item.quantity }}</td>
              <td>{{ formatDate(item.date) }}</td>
              <td>
                <button @click="editProduct(item.id)">Sửa</button>
                <button @click="deleteProduct(item.id)">Xóa</button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
  
      <div class="right-panel" v-if="isFormVisible">
        <h3>Thêm sản phẩm</h3>
        <form @submit.prevent="handleSubmit">
          <div>
            <label for="">Tên sản phẩm</label>
            <input v-model="productInfo.name" type="text" />
          </div>
          <div>
            <label for="">Hình ảnh (url)</label>
            <input v-model="productInfo.imageUrl" type="text" />
          </div>
          <div>
            <label for="">Giá</label>
            <input v-model="productInfo.price" type="number" />
          </div>
          <div>
            <label for="">Số lượng</label>
            <input v-model="productInfo.quantity" type="number" />
          </div>
          <button type="button" @click="resetForm">Đóng</button>
          <button type="submit">Lưu</button>
        </form>
      </div>
    </div>
  </template>
  
  <script setup>
  import { ref, reactive, onMounted } from 'vue';
  
  // Hiển thị danh sách sản phẩm và form
  const isFormVisible = ref(false);
  const products = ref([]);
  const productInfo = reactive({
    id: null,
    name: '',
    imageUrl: '',
    price: 0,
    quantity: 0,
    date: new Date(),
  });
  
  // Hàm định dạng ngày
  const formatDate = (date) => {
    const d = new Date(date);
    return d.toLocaleDateString('vi-VN');
  };
  
  // Hàm lấy dữ liệu sản phẩm từ API (db.json)
  const loadProducts = async () => {
    try {
      const response = await fetch('http://localhost:3000/products');
      products.value = await response.json();
    } catch (error) {
      console.error("Failed to fetch products:", error);
    }
  };
  
  // Hàm thêm/sửa sản phẩm (tương tác với db.json)
  const handleSubmit = async () => {
    if (productInfo.id === null) {
      // Thêm sản phẩm mới (POST)
      await fetch('http://localhost:3000/products', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(productInfo),
      });
    } else {
      // Sửa sản phẩm (PUT)
      await fetch(`http://localhost:3000/products/${productInfo.id}`, {
        method: 'PUT',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(productInfo),
      });
    }
    loadProducts();
    resetForm();
    isFormVisible.value = false;
  };
  
  // Hàm reset form
  const resetForm = () => {
    productInfo.id = null;
    productInfo.name = '';
    productInfo.imageUrl = '';
    productInfo.price = 0;
    productInfo.quantity = 0;
    productInfo.date = new Date();
  };
  
  // Hàm xóa sản phẩm (DELETE)
  const deleteProduct = async (id) => {
    await fetch(`http://localhost:3000/products/${id}`, {
      method: 'DELETE',
    });
    loadProducts();
  };
  
  // Hàm chỉnh sửa sản phẩm
  const editProduct = async (id) => {
    const response = await fetch(`http://localhost:3000/products/${id}`);
    const product = await response.json();
    if (product) {
      productInfo.id = product.id;
      productInfo.name = product.name;
      productInfo.imageUrl = product.imageUrl;
      productInfo.price = product.price;
      productInfo.quantity = product.quantity;
      productInfo.date = product.date;
      isFormVisible.value = true;
    }
  };
  
  // Khi component được mounted, load dữ liệu từ db.json
  onMounted(() => {
    loadProducts();
  });
  </script>
  
  <style>
  .container {
    display: flex;
    gap: 20px;
  }
  .left-panel {
    width: 60%;
  }
  .right-panel {
    width: 35%;
  }
  table {
    width: 100%;
    border-collapse: collapse;
  }
  table th, table td {
    padding: 8px;
    text-align: left;
  }
  button {
    margin-right: 10px;
  }
  </style>  