<template>
  <div id="goods" class="container">
    <!-- loading 套件 start -->
    <loading :active.sync="isLoading"></loading>
    <AlertMessage></AlertMessage>
    <!-- banner -->
    <div class="bannerbox">
      <div class="banner mb-4  mt-3 mt-lg-0" :style="{backgroundImage: `url(${img.banner})` }">
      </div>
      <h2 class="mb-0">產品列表</h2>
    </div>
    <!-- main -->
    <div class="row">
      <div class="col-sm-3 goodstype mb-3 mb-sm-0">
        <h5 class="text-center title">
          <i class="fas fa-puzzle-piece d-none d-md-inline-block"></i>
          產品類別
        </h5>
        <ul class="list-group text-center">
          <li class="list-group-item">
            <a href="#"
            :class="{ active: filterType === 'All' }"
            @click.prevent="getFilter(filterType = 'All')">全部</a>
          </li>
          <li class="list-group-item">
            <a href="#"
            :class="{ active: filterType === 'Travel' }"
            @click.prevent="getFilter(filterType = 'Travel')">旅行</a>
          </li>
          <li class="list-group-item">
            <a href="#"
            :class="{ active: filterType === 'Nature' }"
            @click.prevent="getFilter(filterType = 'Nature')">自然</a>
          </li>
          <li class="list-group-item">
            <a href="#"
            :class="{ active: filterType === 'Flowers' }"
            @click.prevent="getFilter(filterType = 'Flowers')">花卉</a>
          </li>
          <li class="list-group-item">
            <a href="#"
            :class="{ active: filterType === 'Animals' }"
            @click.prevent="getFilter(filterType = 'Animals')">動物</a>
          </li>
          <li class="list-group-item">
            <a href="#"
            :class="{ active: filterType === 'Custom' }"
            @click.prevent="getFilter(filterType = 'Custom')">客製</a>
          </li>
        </ul>
      </div>
      <div class="col-sm-9">
        <div class="row">
          <div class="col-md-6 mb-4" v-for="item in newProducts" :key="item.id">
            <div class="card shadow-sm h-100">
              <div class="card-head">
                <router-link :to="`/goodsdetail/${item.id}`">
                  <img :src="item.imageUrl[0]" :alt="item.title"
                  class="w-100 card-img-top img-md-fluid">
                </router-link>
                <div
                  title="收藏"
                  class="iconTags"
                  @click.prevent="addFollow(item.id)">
                  <i
                    v-if="followData.indexOf(item.id) === -1"
                    class="far fa-heart">
                  </i>
                  <i v-else class="fas fa-heart"></i>
                </div>
                <div class="info">
                  <router-link :to="`/goodsdetail/${item.id}`" class="detaillogo">
                  <i class="fas fa-info"></i>
                  </router-link>
                  <a class="cartlogo" @click.prevent="addToCart(item.id)">
                    <i class="fas fa-cart-arrow-down"></i>
                  </a>
                </div>
              </div>
              <router-link :to="`/goodsdetail/${item.id}`" class="text-decoration-none card-body">
                <h6>{{ item.title }}</h6>
                <div class="d-flex justify-content-between">
                  <span class="text-secondary">
                    <del>原價：{{ item.origin_price | currency }}</del
                  ></span>
                  <span class="text-danger h5">特價：{{ item.price | currency }}</span>
                </div>
              </router-link>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import AlertMessage from '@/components/AlertMessage.vue';

export default {
  name: 'Goods',
  components: {
    AlertMessage,
  },
  data() {
    return {
      products: [],
      newProducts: [],
      isLoading: false,
      filterType: 'All',
      img: {
        banner:
          'https://hexschool-api.s3.us-west-2.amazonaws.com/custom/JmgReiZHDg7yjqzHTiUaZFBtOVdJVI4Zlpaeiygc1pwPwwxW6udpRZMMZIUczHFAJramsTDIuwnWgIgboFgL31wgkqpB2D6YmQqznxaKDU1vF95zSTZlR5peeKiKXGvo.jpeg',
      },
      followData: JSON.parse(localStorage.getItem('followCard')) || [],
    };
  },
  methods: {
    getProducts() {
      const vm = this;
      const api = `${process.env.VUE_APP_APIPATH}/api/${process.env.VUE_APP_UUID}/ec/products`;
      vm.isLoading = true;
      vm.$http.get(api).then((res) => {
        vm.products = res.data.data;
        vm.isLoading = false;
        vm.getFilter();
      });
    },
    getFilter() {
      switch (this.filterType) {
        case 'All':
          this.newProducts = this.products;
          break;
        case 'Travel':
          this.newProducts = this.products.filter((item) => item.category === 'Travel');
          break;
        case 'Nature':
          this.newProducts = this.products.filter((item) => item.category === 'Nature');
          break;
        case 'Flowers':
          this.newProducts = this.products.filter((item) => item.category === 'Flower');
          break;
        case 'Animals':
          this.newProducts = this.products.filter((item) => item.category === 'Animal');
          break;
        case 'Custom':
          this.newProducts = this.products.filter((item) => item.category === 'Customize');
          break;
        default:
          break;
      }
    },
    addToCart(id, quantity = 1) {
      const vm = this;
      const url = `${process.env.VUE_APP_APIPATH}/api/${process.env.VUE_APP_UUID}/ec/shopping`;
      const cart = {
        product: id,
        quantity,
      };
      vm.isLoading = true;
      vm.$http.post(url, cart).then(() => {
        vm.isLoading = false;
        vm.$bus.$emit('message:push',
          '加入購物車成功', 'success');
        vm.$bus.$emit('getcart');
      }).catch((error) => {
        vm.isLoading = false;
        const errorData = error.response.data.errors[0];
        vm.$bus.$emit('message:push',
          `${errorData}`, 'danger');
      });
    },
    addFollow(id) {
      const vm = this;
      const followId = this.followData.indexOf(id);
      if (followId === -1) {
        vm.followData.push(id);
        vm.$bus.$emit('message:push',
          '加入收藏', 'success');
      } else {
        vm.followData.splice(followId, 1);
        vm.$bus.$emit('message:push',
          '取消收藏', 'danger');
      }
      localStorage.setItem('followCard', JSON.stringify(this.followData));
    },
  },
  created() {
    this.getProducts();
  },

};
</script>
