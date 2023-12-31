<template>
  <div class="container py-4">
    <span class="spinner-border" v-if="!product"></span>
    <div class="row" v-else>
      <div class="col col-12 col-md-6">
        <div style="display: flex; gap: 5px">
          <div style="flex: 1">
            <img
              v-if="product.images[imgIndex]"
              :src="fileService.getFileUrl(product.images[imgIndex].path)"
              :alt="product.images[imgIndex].ten"
              style="width: 100%; aspect-ratio: 1; object-fit: cover"
              class="rounded border"
            />
          </div>

          <div
            class="d-flex gap-2"
            style="flex-direction: column; overflow: auto; max-height: 450px"
          >
            <img
              v-for="(image, index) in product.images"
              :key="image.id"
              :src="fileService.getFileUrl(image.path)"
              :alt="image.ten"
              style="width: 150px; aspect-ratio: 1; cursor: pointer"
              class="border rounded border-2"
              :class="{
                'border-primary': imgIndex === index
              }"
              @click="imgIndex = index"
            />
          </div>
        </div>
      </div>

      <div class="col col-12 col-md-6">
        <h2>{{ product.ten }}</h2>

        <p>
          {{ product.mota }}
        </p>

        <div class="d-flex gap-1 mb-3">
          <button class="btn btn-outline-primary" disabled>
            <i class="fa-solid fa-dollar-sign"></i>
            {{ vndFormat(product.gia) }}
          </button>

          <button class="btn" disabled>Còn lại: {{ product.soluong }}</button>
        </div>

        <label class="form-label" v-if="user">Đặt hàng</label>
        <input
          class="form-control mb-2"
          placeholder="Số lượng"
          type="number"
          v-model="quantity"
          min="1"
          :max="product.soluong"
          v-if="user"
        />

        <a href="#" @click="addToCart" class="btn btn-secondary d-block mb-2" v-if="user">
          <i class="fa fa-cart-plus"> </i>
          Thêm vào giỏ hàng
        </a>

        <a href="#" class="btn btn-primary d-block" @click="buy" v-if="user"> Mua </a>

        <div v-if="cartItem && user">
          Đã có <span style="font-weight: bold">{{ cartItem.soluong }}</span> sản phẩm này trong giỏ
          hàng
        </div>
      </div>
    </div>

    <h3 class="my-3">Các sản phẩm khác</h3>

    <div class="row">
      <div
        class="col col-12 col-md-4 col-lg-3 col-xl-2 mb-2 p-1"
        v-for="product in otherProducts"
        :key="product.id"
      >
        <ProductCard :product="product" />
      </div>
    </div>
  </div>
</template>

<script>
import ProductCard from '@/components/ProductCard.vue'
import fileService from '@/services/file.service'
import hanghoaService from '@/services/hanghoa.service'
import { useCartStore } from '@/stores/cart'
import { useUserStore } from '@/stores/user'
import vndFormat from '@/utils/vndFormat'
import { ref } from 'vue'

const cartStore = useCartStore()
const userStore = useUserStore()

export default {
  name: 'ProductView',
  data() {
    const product = ref(null)
    const imgIndex = ref(0)
    const quantity = ref(1)
    const cartItem = ref(null)
    const user = ref(userStore.user)
    const otherProducts = ref([])
    return { product, fileService, imgIndex, quantity, cartItem, otherProducts, user }
  },
  watch: {
    product(value) {
      if (!value) return
      this.cartItem = cartStore.getItemById(value.id)
    },
    cartStore() {
      if (!this.product) return
      this.cartItem = cartStore.getItemById(this.product.id)
    },
    $route() {
      this.getProductData()
    }
  },
  methods: {
    async getProductData() {
      try {
        const { id } = this.$route.params
        const res = await hanghoaService.getById(id)
        this.product = res
      } catch (error) {
        console.log(error)
      }
    },
    addToCart() {
      cartStore.addToCart({
        id: this.product.id,
        soluong: this.quantity
      })
      this.cartItem = cartStore.getItemById(this.product.id)
      this.$toast.success('Thêm vào giỏ hàng thành công!', {})
    },
    vndFormat,
    async getOtherProducts() {
      try {
        this.otherProducts = (await hanghoaService.getAll()).data
      } catch (error) {
        this.$toast.error(error.message)
      }
    },
    buy() {
      this.addToCart()
      this.$router.push('/cart')
    },
  },
  beforeMount() {
    this.getProductData()
    this.getOtherProducts()
  },
  components: { ProductCard }
}
</script>
