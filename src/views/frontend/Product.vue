<template>
  <div>
    <div class="product-banner">
      <h2 class="text-center">商品介紹</h2>
    </div>
    <div class="container custom-container mt-3">
      <loading :active.sync="isLoading">
        <i class="loading-box"></i>
      </loading>
      <nav aria-label="breadcrumb">
        <ol class="breadcrumb">
          <li class="breadcrumb-item">
            <router-link to="/">Home</router-link>
          </li>
          <li class="breadcrumb-item">
            <router-link to="/products">全部商品</router-link>
          </li>
          <li class="breadcrumb-item active" aria-current="page">
            {{ product.title }}
          </li>
        </ol>
      </nav>
      <section class="d-flex row product-box my-4">
        <div class="more col-lg-6 col-md-12">
          <img
            @click="openDialog"
            :src="product.imageUrl"
            alt=""
            class="product-img"
            title="點擊看更多放大美圖"
          />
          <v-dialog v-model="dialog" max-width="600px">
            <v-card>
              <v-card-title class="headline"><b>放大預覽</b></v-card-title>
              <v-card-text>
                <!-- <v-img :src="item" class="image-full"></v-img> -->
                <v-carousel show-arrows="hover" hide-delimiter-background :key="componentKey">
                  <v-carousel-item
                    v-for="(item, i) in imgs"
                    :key="item[i]"
                    :src="item"
                    cover
                  ></v-carousel-item>
                </v-carousel>
              </v-card-text>
              <v-card-actions>
                <v-btn color="red darken-1" text @click="closeDialog"
                  >關閉
                  <i class="fa-solid fa-xmark fa-2x"></i>
                </v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>
        </div>
        <div class="col-lg-6 col-md-12 pro-content-box">
          <div>
            <div class="d-flex justify-content-between align-items-baseline">
              <h4 class="pt-3 text-left" id="exampleModalLabel">
                {{ product.title }}
              </h4>
              <div
                v-if="followData.indexOf(product.id) === -1"
                class="tags-size"
                @click.prevent="addFollow(product.id)"
              >
                <i class="far fa-bookmark"> </i>
              </div>
              <div v-else class="tags-size" @click.prevent="addFollow(product.id)">
                <i class="fas fa-bookmark"></i>
              </div>
            </div>
            <hr />
            <p class="mb-2">{{ product.description }}</p>
          </div>
          <div class="text-right">
            <span class="product-origin-price pr-2" v-if="product.origin_price !== 0">
              {{ $filters.currency(product.origin_price) }}
            </span>
            <span class="pr-2 product-price">
              {{ $filters.currency(product.price) }}
            </span>
          </div>
          <select name class="form-control my-3" v-model="product.num">
            <option value="0" disabled selected>請選擇數量</option>
            <option :value="num" v-for="num in 10" :key="num">{{ num }} {{ product.unit }}</option>
          </select>
          <div class="product-footer mb-3">
            <div class="subtotal-col text-nowrap mr-3">
              小計 <span class="text-lightgary">| </span>
              <strong>{{ $filters.currency(product.num * product.price) }}</strong>
            </div>
            <button
              v-if="status.loadingItem === product.id"
              class="btn btn-shopping btn-sm"
              type="button"
              :disabled="status.loadingItem === product.id"
              @click="addToCart(product.id, product.num)"
            >
              <i class="fas fa-spinner fa-spin"></i>
              加到購物車
            </button>
            <button
              v-else
              class="btn btn-shopping btn-sm"
              type="button"
              @click="addToCart(product.id, product.num)"
            >
              加到購物車
            </button>
          </div>
        </div>
      </section>
      <v-card class="mb-4" color="gray" variant="tonal">
        <v-tabs v-model="tab" align-tabs="left" color="deep-purple-accent-4">
          <v-tab :value="1"><b>商品介紹</b></v-tab>
          <v-tab :value="2"><b>購物須知</b></v-tab>
          <v-tab :value="3"><b>退換貨說明</b></v-tab>
        </v-tabs>
        <v-tabs-window v-model="tab">
          <v-tabs-window-item :value="1">
            <v-container fluid><p v-html="product.content" style="color: black"></p></v-container
          ></v-tabs-window-item>
          <v-tabs-window-item :value="2"
            ><v-container fluid><p v-html="profile" style="color: black"></p></v-container>
          </v-tabs-window-item>
          <v-tabs-window-item :value="3">
            <v-container fluid><p v-html="refund" style="color: black"></p></v-container>
          </v-tabs-window-item>
        </v-tabs-window>
      </v-card>
      <section class="row mb-4">
        <div class="col-md-12">
          <h5 class="font-weight-bold mb-3 h5">相關商品</h5>
          <Sametype :product="product" @update="getProduct" />
        </div>
      </section>
    </div>
  </div>
</template>

<script>
import Toast from "@/alert/Toast";
import Sametype from "@/components/frontend/Sametype.vue";

export default {
  components: {
    Sametype,
  },
  data() {
    return {
      tab: null,
      imgs: [],
      dialog: false,
      componentKey: 0,
      product: {
        num: 1,
      },
      id: "",
      isLoading: false,
      status: {
        loadingItem: "",
      },
      profile: `<ul>
                  <li>每次購物滿 1299 元以上免運費，未達 1299 元 需加付 80 元運費。</li>
                  <li>出貨時間為 2-3。大型商品(如：帳篷等)以貨運公司配送。</li>
                  <li>貨品均一年保固免費維修。</li>
                </ul>`,
      refund: `<p>
                網路購物的消費者，享有商品貨到日起七天猶豫期。<br />
                但猶豫期並非試用期，所以，您所退回的商品必須是全新的狀態。
              </p>`,
      followData: JSON.parse(localStorage.getItem("followCard")) || [],
    };
  },
  created() {
    this.id = this.$route.params.productId;
    this.getProduct();
  },
  watch: {
    $route() {
      this.id = this.$route.params.productId;
      this.getProduct();
    },
  },
  methods: {
    openDialog() {
      this.dialog = true;
    },
    closeDialog() {
      this.dialog = false;
    },
    addLineBreaks(text) {
      let updatedText = text.replace(/\n/g, "<br />");
      // 分割字串
      let sentences = updatedText.split(".");
      // 去掉空白字串
      sentences = sentences.filter((sentence) => sentence.trim() !== "");
      // 連接字串並加入換行符號
      return sentences.join(".<br />");
    },
    getProduct() {
      if (this.imgs.length > 0) {
        this.imgs = [];
      }
      const api = `${process.env.VUE_APP_API}api/${process.env.VUE_APP_PATH}/product/${this.id}`;
      this.isLoading = true;
      this.$http
        .get(api)
        .then((response) => {
          this.isLoading = false;
          if (response.data.success === true) {
            this.product = response.data.product;
            if ("content" in this.product) {
              this.product.content = this.addLineBreaks(this.product.content);
            } else {
              this.product.content = "此產品暫無資訊";
            }
            this.product.num = 1;
            if ("imagesUrl" in this.product && "imageUrl" in this.product) {
              this.imgs = this.product.imagesUrl;
              this.imgs.unshift(this.product.imageUrl);
            } else if (
              "imageUrl" in this.product &&
              !("imagesUrl" in this.product) &&
              this.imgs.length === 0
            ) {
              this.imgs.push(this.product.imageUrl);
            } else if ("imagesUrl" in this.product && !("imageUrl" in this.product)) {
              this.imgs = this.product.imagesUrl;
            }
            this.componentKey += 1;
          } else {
            Toast.fire({
              title: `${response.data.message}`,
              icon: "warning",
            });
            this.$router.replace("/products");
          }
        })
        .catch(() => {
          this.isLoading = false;
        });
    },
    addToCart(id, num) {
      this.status.loadingItem = id;
      const url = `${process.env.VUE_APP_API}api/${process.env.VUE_APP_PATH}/cart`;
      const cart = {
        product_id: id,
        qty: num,
      };
      this.$http
        .post(url, { data: cart })
        .then((response) => {
          setTimeout(() => {
            this.status.loadingItem = "";
            this.$emitter.emit("update-total"); // 更新購物車數量
            Toast.fire({
              title: "已加入購物車",
              icon: "success",
            });
          }, 500);
        })
        .catch((err) => {
          this.status.loadingItem = "";
          Toast.fire({
            title: `${err.response.data.errors}`,
            icon: "warning",
          });
        });
    },
    addFollow(id) {
      const followId = this.followData.indexOf(id);
      if (followId === -1) {
        this.followData.push(id);
        Toast.fire({
          title: "已加入收藏",
          icon: "success",
        });
      } else {
        this.followData.splice(followId, 1);
        Toast.fire({
          title: "已取消收藏",
          icon: "success",
        });
      }
      localStorage.setItem("followCard", JSON.stringify(this.followData));
    },
  },
};
</script>

<style lang="scss" scoped>
@media (max-width: 1290px) {
  .custom-container {
    padding-left: 60px;
    padding-right: 60px;
  }
}
@media (max-width: 1200px) {
  .custom-container {
    padding-left: 30px;
    padding-right: 30px;
  }
}
@media (max-width: 992px) {
  .custom-container {
    padding-left: 15px;
    padding-right: 15px;
  }
  .pro-content-box {
    width: 100%;
  }
  .product-box {
    margin: 0 8em !important;
  }
}
@media (max-width: 768px) {
  .custom-container {
    padding-left: 0px;
    padding-right: 0px;
  }
  .product-box {
    margin: 0 4em !important;
  }
}
@media (max-width: 568px) {
  .product-box {
    margin: 0 2em !important;
  }
}
</style>
