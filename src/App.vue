<template>
  <div>
    <el-row justify="center" :gutter="20">
      <el-button type="primary" @click="scrapeProducts">Scrape Data</el-button>
      <el-button type="danger" @click="deleteProducts">Delete All Products</el-button>
    </el-row>
    <el-divider></el-divider>
    <el-row justify="center">
      <h1>Products</h1>
    </el-row>
    <el-row justify="center" v-if="products?.length === 0">
      <el-empty description="No products available." />
    </el-row>
    <el-row :gutter="20">
      <el-col :span="6" v-for="product in products" :key="product._id">
        <el-card shadow="hover">
          <img :src="product.image" alt="Product Image" style="width: 100%;" />
          <h2>{{ product.name }}</h2>
          <p v-if="product.price.includes('Not')">{{ product.price }}</p>
          <p v-else>{{ product.price }} EUR</p>
          <p>Rating :{{ product.rating }}</p>
          <p>Reviews Count :{{ product.ratingCount }}</p>
          <!-- Offers Section -->
          <div v-if="product.offers.length > 0" class="offers-section">
            <h3>Offers:</h3>
            <ul>
              <li v-for="offer in product.offers" :key="offer._id">
                <p>Price: {{ offer.priceOffer }} EUR</p>
                <p>Availability: {{ offer.availability }}</p>
                <p>Shipping:</p>
                <ul>
                  <li v-for="shipping in offer.shippings" :key="shipping._id">
                    <p>Type: {{ shipping.type }}</p>
                    <p>Duration: {{ shipping.duration }}</p>
                  </li>
                </ul>
              </li>
            </ul>
          </div>
        </el-card>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import axios from 'axios';
import { ElLoading, ElMessage } from 'element-plus';

export default {
  data() {
    return {
      products: [],
      loading: null, 
    };
  },
  methods: {
    async fetchProducts() {
      this.loading = ElLoading.service({ text: 'Loading products...' });
      try {
        const response = await axios.get('http://localhost:3000/products');
        this.products = response.data;
        if(this.products.length == 0){
          ElMessage.info('No products');
        }else{
          ElMessage.success('Products fetched successfully!');
        }
      } catch (error) {
        console.error('Error fetching products:', error);
        ElMessage.error('Failed to fetch products.');
      } finally {
        this.loading.close();
      }
    },
    async scrapeProducts() {
      this.loading = ElLoading.service({ text: 'Scraping products...' });
      try {
        const response = await axios.post('http://localhost:3000/scrape');
        this.fetchProducts();
        ElMessage.success('Products scraped successfully!');
      } catch (error) {
        console.error('Error scraping products:', error);
        ElMessage.error('Failed to scrape products.');
      } finally {
        this.loading.close();
      }
    },
    async deleteProducts() {
      this.loading = ElLoading.service({ text: 'Deleting products...' });
      try {
        const response = await axios.delete('http://localhost:3000/products');
        this.products = [];
        ElMessage.success('Products deleted successfully!');   
      } catch (error) {
        console.error('Error deleting products:', error);
        ElMessage.error('Failed to delete products.');
      } finally {
        this.loading.close();
      }
    }
  },
  mounted() {
    this.fetchProducts();
  }
};


</script>