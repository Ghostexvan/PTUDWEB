<template>
  <!-- <div class="admin-pannel" v-if="isAdmin">
    <RouterLink class="btn btn-sm btn-sm btn-primary" to="/admin">
      <i class="fa fa-shield"></i>
      ADMIN CP
    </RouterLink>
  </div> -->
  <nav class="navbar navbar-expand-lg border-bottom bg-black" v-if="isAdmin || $route.path.localeCompare('/authorize_admin') === 0" style="position: sticky; top: 0">
    <div class="container px-4 px-lg-5">
      <RouterLink class="navbar-brand text-white" to="/admin">Hobby Store Admin</RouterLink>
      <button
        class="navbar-toggler"
        type="button"
        data-bs-toggle="collapse"
        data-bs-target="#navbarSupportedContent"
        aria-controls="navbarSupportedContent"
        aria-expanded="false"
        aria-label="Toggle navigation"
      >
        <span class="navbar-toggler-icon"></span>
      </button>
      <div class="collapse navbar-collapse" id="navbarSupportedContent">
        <ul class="navbar-nav me-auto mb-2 mb-lg-0 ms-lg-4"></ul>
        <div class="d-flex gap-1">
          <div class="dropdown" :class="{ show: showModal }" v-if="isLogged" @click="toggleModal">
            <button class="btn btn-secondary dropdown-toggle bg-black text-white" type="button">
              {{ user?.ten }}
            </button>
            <ul class="dropdown-menu" :class="{ show: showModal }">
              <li>
                <RouterLink class="dropdown-item" to="/customer">
                  <i class="fa fa-user"></i>
                  Thông tin tài khoản
                </RouterLink>
              </li>
              <li>
                <a class="dropdown-item" href="#" @click="logout">
                  <i class="fa-solid fa-arrow-right-from-bracket"></i>
                  Đăng xuất</a
                >
              </li>
            </ul>
          </div>
        </div>
      </div>
    </div>
  </nav>

  <nav class="navbar navbar-expand-lg border-bottom bg-white" v-else style="position: sticky; top: 0">
    <div class="container px-4 px-lg-5">
      <RouterLink class="navbar-brand" to="/">Hobby Store</RouterLink>
      <button
        class="navbar-toggler"
        type="button"
        data-bs-toggle="collapse"
        data-bs-target="#navbarSupportedContent"
        aria-controls="navbarSupportedContent"
        aria-expanded="false"
        aria-label="Toggle navigation"
      >
        <span class="navbar-toggler-icon"></span>
      </button>
      <div class="collapse navbar-collapse" id="navbarSupportedContent">
        <ul class="navbar-nav me-auto mb-2 mb-lg-0 ms-lg-4"></ul>
        <div class="d-flex gap-1">
          <RouterLink class="btn" to="/cart" v-if="isAdmin === false && user">
            <i class="fa fa-cart-shopping"></i>
            <span class="badge bg-danger text-white ms-1 rounded-pill">{{ numberOfItem }}</span>
          </RouterLink>

          <div class="dropdown" :class="{ show: showModal }" v-if="isLogged" @click="toggleModal">
            <button class="btn btn-secondary dropdown-toggle" type="button">
              {{ user?.ten }}
            </button>
            <ul class="dropdown-menu" :class="{ show: showModal }">
              <li>
                <RouterLink class="dropdown-item" to="/customer">
                  <i class="fa fa-user"></i>
                  Thông tin tài khoản
                </RouterLink>
              </li>
              <li>
                <a class="dropdown-item" href="#" @click="logout">
                  <i class="fa-solid fa-arrow-right-from-bracket"></i>
                  Đăng xuất</a
                >
              </li>
            </ul>
          </div>

          <div class="d-flex gap-1" v-else-if="$route.path !== '/auth/login' && $route.path !== '/auth/register'">
            <RouterLink to="/auth/login" >
              <button class="btn btn-primary">Đăng nhập</button>
            </RouterLink>

            <RouterLink to="/auth/register">
              <button class="btn btn-primary">Đăng ký</button>
            </RouterLink>
          </div>
          
        </div>
      </div>
    </div>
  </nav>
</template>

<script>
import { useCartStore } from '@/stores/cart'
import { useUserStore } from '@/stores/user'
import { ref } from 'vue'
import { RouterLink } from 'vue-router'

export default {
  components: {
    RouterLink
  },

  data() {
    const cartStore = useCartStore()
    const userStore = useUserStore()
    const showModal = ref(false)
    const cart = ref(cartStore.cart)
    return {
      userStore,
      cartStore,
      cart,
      showModal
    }
  },
  watch: {
    cartStore() {
      this.cart = this.cartStore.cart
    },
    cart() {}
  },
  methods: {
    toggleModal(event) {
      event.stopPropagation()

      this.showModal = !this.showModal
    },
    onWindowClick() {
      this.showModal = false
    },
    logout() {
      this.userStore.logout().then(() => {
        this.$router.push('/')
      })
    },
  },
  computed: {
    isLogged() {
      return this.userStore.isLogged()
    },
    user() {
      return this.userStore.user
    },
    isStaff() {
      return this.userStore.user?.role === 'nhanvien'
    },
    isAdmin() {
      return this.userStore.isAdmin()
    },
    numberOfItem() {
      console.log(this.cartStore.cart)
      return this.cartStore.cart ? this.cartStore.cart.length : 0
    }
  },
  mounted() {
    window.addEventListener('click', this.onWindowClick)
  },
  unmounted() {
    window.removeEventListener('click', this.onWindowClick)
  }
}
</script>

<style scoped>
.admin-pannel {
  padding: 10px;
  display: flex;
  justify-content: flex-start;
  align-items: center;
  background: #ddd;
}
</style>
