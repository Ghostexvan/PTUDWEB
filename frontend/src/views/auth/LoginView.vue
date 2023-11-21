<template>
  <div class="container py-4">
    <VeeForm
      class="card p-4 shadow shadow-sm"
      @submit="login"
      :validation-schema="validationScheme"
      v-slot="{ errors }"
    >
      <h2 v-if="$route.path !== '/authorize_admin'">Đăng nhập</h2>
      <h2 v-else>Đăng nhập ADMIN</h2>
      <div class="mb-3">
        <label for="username" class="form-label">Tên đăng nhập</label>
        <field
          name="username"
          id="username"
          class="form-control"
          :class="{ 'is-invalid': errors.username }"
          placeholder="Tên đăng nhập"
        />
        <error-message
          name="username"
          class="invalid-feedback animate__animated animate__headShake"
        />
      </div>

      <div class="mb-3">
        <label for="inputPassword" class="col-form-label">Mật khẩu</label>
        <field
          name="password"
          type="password"
          class="form-control"
          :class="{ 'is-invalid': errors.password }"
          id="inputPassword"
          placeholder="Mật khẩu"
        />
        <error-message
          name="password"
          class="invalid-feedback animate__animated animate__headShake"
        />
      </div>

      <button class="btn btn-primary" type="submit">Đăng nhập</button>

      <p class="text text-dark py-2" v-if="$route.path.localeCompare('/authorize_admin') !== 0">
        Bạn chưa có tài khoản?
        <RouterLink to="/auth/register">Đăng ký ngay</RouterLink>
      </p>
    </VeeForm>
  </div>
</template>

<style scoped>
form {
  width: 480px;
  max-width: 100%;
  margin: 0 auto;
}
</style>

<script>
import { Form as VeeForm, Field, ErrorMessage } from 'vee-validate'
import * as yup from 'yup'
import { RouterLink } from 'vue-router'
import authService from '@/services/auth.service'
import { useUserStore } from '@/stores/user'

const validationScheme = yup.object().shape({
  username: yup
    .string()
    .required('Vui lòng nhập tên đăng nhập')
    .min(3, 'Tên đăng nhập phải có ít nhất 3 ký tự')
    .max(10, 'Tên đăng nhập không được dài quá 10 ký tự'),
  password: yup
    .string()
    .required('Vui lòng nhập mật khẩu')
    .min(6, 'Mật khẩu phải có ít nhất 6 ký tự')
    .max(20, 'Mật khẩu không được dài quá 10 ký tự')
})

export default {
  name: 'LoginView',
  components: {
    VeeForm,
    Field,
    ErrorMessage,
    RouterLink
  },
  data() {
    const userStore = useUserStore()

    return {
      validationScheme,
      userStore
    }
  },
  methods: {
    async login(value) {
      try {
        const { accessToken } = await authService.login(value)
        this.userStore.setAccessToken(accessToken)

        const res = await authService.auth()
        if (this.$route.path.localeCompare('/auth/login') === 0 && res.role === "nhanvien"){
          this.logout()
          throw {
            message: 'Mật khẩu bị sai'
          }
        }
        else if (this.$route.path.localeCompare('/authorize_admin') === 0 && res.role !== "nhanvien"){
          this.logoutAdmin()
          throw {
            message: 'Chỉ được đăng nhập với tài khoản có quyền'
          }
        }

        await this.userStore.setUser(res)
        
        if (res.role === "nhanvien")
          this.$router.push('/admin')
        else
          this.$router.push('/')
      } catch (error) {
        this.$toast.error(error.message)
        this.$router.push('/')
      }
    },
    logout() {
      this.userStore.logout().then(() => {
        this.$router.push('/auth/login')
      })
    },
    logoutAdmin() {
      this.userStore.logout().then(() => {
        this.$router.push('/admin')
      })
    }
  }
}
</script>
