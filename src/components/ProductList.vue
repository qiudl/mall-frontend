<template>
  <div>
    <h1>商品列表</h1>
    <ul>
      <li v-for="product in products" :key="product.id">
        {{ product.name }} - {{ product.description }}
        <button v-if="product.status === 'pending'" @click="approveProduct(product.id)">审核通过</button>
      </li>
    </ul>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      products: []
    };
  },
  mounted() {
    this.getPendingProducts();
  },
  methods: {
    getPendingProducts() {
      axios.get('http://localhost:8080/products/pending')
        .then(response => {
          this.products = response.data;
        })
        .catch(error => {
          console.error(error);
        });
    },
    approveProduct(productId) {
      axios.put(`http://localhost:8080/products/${productId}/approve`)
        .then(response => {
          console.log(response.data);
          this.getPendingProducts();
        })
        .catch(error => {
          console.error(error);
        });
    }
  }
};
</script>