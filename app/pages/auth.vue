<template>
  <div>認証処理中です</div>
</template>

<script>
export default {
  head() {
    return {
      title: '認証'
    }
  },
  fetch({ store }) {
    store.dispatch('header/unsetAllowLogin')
  },
  mounted() {
    setTimeout(async () => {
      const token = this.$route.query.token
      if (token) {
        this.$axios.setToken(token, 'Bearer')
        await this.$axios
          .put('/api/sessions/token')
          .then(res => {
            this.$store.dispatch('user/login', res.data.token)
          })
          .catch(error => {
            console.log(error.message)
            this.$store.dispatch('user/logout', token)
            this.$router.replace({ path: '/' })
          })

        const newToken = this.$store.getters['user/getAuthToken']
        this.$axios.setToken(newToken, 'Bearer')
        const res = await this.$axios.get('/api/users/me').catch(error => {
          console.log(error.message)
          this.$store.dispatch('user/logout', newToken)
          this.$router.replace({ path: '/' })
        })
        console.log(res)
        this.$store.dispatch('user/load', res.data)
      }
      this.$router.replace({ path: '/mypage' })
    }, 0)
  }
}
</script>

<style lang="postcss" scoped>
* {
  display: none;
}
</style>
