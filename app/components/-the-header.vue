<template>
  <nav>
    <div class="header">
      <img src="@/assets/img/logo.svg" class="svg logo" />
      <nuxt-link class="no-underline" to="/">Diplomacy MOE</nuxt-link>
    </div>
    <div v-if="!authToken && allowLogin" class="pt-1">
      <nuxt-link
        id="header-login-link"
        class="link btn-header-twitter-login"
        to="/login"
      >
        <span id="header-twitter-logo"></span>
        ログイン
      </nuxt-link>
    </div>
    <div v-if="authToken" class="pt-2">
      <div @click="openMenu" class="cursor-pointer">
        <img id="header-user-image" :src="imageUrl" />
        <span class="dropdown-caret"></span>
      </div>
      <div v-if="isMenuActive" class="menu">
        <div class="caret">
          <span class="caret-outer"></span>
          <span class="caret-inner"></span>
        </div>
        <ul>
          <li @click="showMypage" class="link">マイページ</li>
          <li @click="clickLogout" class="link">ログアウト</li>
          <li
            @click="toggleFakePowerSelector"
            v-if="isAdmin"
            :class="{ checked: isFakePowerSelectorActive }"
            class="link"
          >
            国選択メニュー表示
          </li>
        </ul>
      </div>
    </div>
    <div v-if="isFakePowerSelectorActive" class="fake-power-selector">
      TODO
    </div>
    <div v-if="isMenuActive" @click="closeMenu" class="dropdown-bg"></div>
  </nav>
</template>

<script>
import { mapState, mapActions } from 'vuex'

export default {
  data: function() {
    return {
      offsetY: 0
    }
  },
  computed: {
    ...mapState('user', ['authToken', 'imageUrl', 'isAdmin']),
    ...mapState('header', ['allowLogin']),
    ...mapState('layout', ['isMenuActive', 'isFakePowerSelectorActive'])
  },
  methods: {
    ...mapActions('user', ['logout']),
    ...mapActions('layout', ['closeFakePowerSelector']),
    openMenu: async function() {
      this.offsetY =
        document.documentElement.scrollTop || document.body.scrollTop
      await this.$store.dispatch('layout/openMenu')
      document.getElementById('__content').scroll(0, this.offsetY)
    },
    closeMenu: async function() {
      await this.$store.dispatch('layout/closeMenu')
      window.scroll(0, this.offsetY)
    },
    clickLogout: function() {
      this.closeMenu()
      this.closeFakePowerSelector()
      this.logout()
      this.$router.push('/logout')
    },
    showMypage: function() {
      this.closeMenu()
      this.$router.push('/mypage')
    },
    toggleFakePowerSelector: function() {
      this.closeMenu()
      this.$store.dispatch('layout/toggleFakePowerSelector')
    }
  }
}
</script>

<style lang="postcss" scoped>
nav {
  @apply flex justify-between flex-wrap;
  @apply px-3 pb-1;
  @apply bg-gray-400 text-gray-700 text-sm;
  @apply fixed w-full z-50;

  & .header {
    @apply text-lg pt-2;

    & img.logo {
      @apply inline-block;
      width: 24px;
    }

    & a {
      @apply text-gray-800;
    }
  }

  & a.link {
    @apply cursor-pointer;
  }

  & a.btn-header-twitter-login {
    @apply inline-block mt-1 py-0 pl-1 pr-2 rounded-sm;
    @apply text-white text-xs;
    background-color: #55acee;
    line-height: 1.5rem;

    &:hover {
      @apply bg-blue-700;
    }

    & #header-twitter-logo {
      @apply w-6 h-6 inline-block align-bottom;
      width: 16px;
      background: url('~assets/img/twitter_logo.png') center no-repeat;
      background-size: contain;
    }
  }

  & #header-user-image {
    @apply w-6 h-6 rounded-full inline-block mr-2 align-bottom;
  }
}
</style>

<style lang="postcss" scoped>
.dropdown-caret {
  @apply border-gray-700;
  @apply border-solid border-b-0 h-0 w-0 inline-block align-middle;
  border-left: 4px solid transparent;
  border-right: 4px solid transparent;
  border-top-width: 4px;
}
.caret {
  @apply border-white;
  @apply absolute right-0 mr-2;
  @apply border-solid border-b-0 h-0 w-0 inline-block align-middle;
  top: -6px;
  border-left: 8px solid transparent;
  border-right: 8px solid transparent;
  border-top-width: 8px;
  transform: rotate(180deg);
}
.menu {
  @apply w-64;
  @apply border border-solid rounded bg-white shadow-md;
  @apply py-2 m-3;
  @apply absolute left-auto right-0 z-20;
  border-left: 1px #edf2f7 solid;
  border-right: 1px #edf2f7 solid;
}
.menu ul {
  @apply p-0 z-30 w-full list-none;
}
.menu ul li {
  @apply w-full py-1 bg-transparent;
  @apply pl-10 pr-5 text-xl;
}
.menu ul li a {
  @apply text-gray-700;
}
.menu ul li:hover {
  @apply bg-blue-700 text-gray-200;
}
ul li.checked {
  position: relative;
}
ul li.checked::after {
  display: block;
  content: '';
  position: absolute;
  top: 0.8em;
  left: 0.8em;
  width: 10px;
  height: 6px;
  border-left: 2px solid #3498db;
  border-bottom: 2px solid #3498db;
  -webkit-transform: rotate(-45deg);
  transform: rotate(-45deg);
}
.fake-power-selector {
  @apply fixed w-full inset-x-0 block;
  @apply bg-black text-white;
  @apply h-8;
  @apply text-center leading-loose;
  top: 39.5px;
}
.dropdown-bg {
  @apply w-full h-screen fixed inset-0 z-10;
}
</style>

<style lang="postcss" scoped>
@screen sm {
  nav {
    @apply px-6;
  }
  .logo {
    width: 32px;
  }
  .header {
    @apply text-xl pt-1;
  }
  .menu {
    @apply w-48;
  }
  .menu ul li {
    @apply px-3 text-base;
  }
}
</style>
