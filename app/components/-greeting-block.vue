<template>
  <div class="greeting-block">
    <div class="greeting">
      <h1>
        Diplomacy MOE へようこそ！
      </h1>
      <p>
        Diplomacy MOE は名作ボードゲームディプロマシーを
        オンラインで遊べるように実装したものです。
      </p>
      <p>
        7 人の参加者はそれぞれ 20 世紀初頭の欧州列強各国の指導者となり、
        外交手腕を発揮して欧州の覇権を争います。
      </p>
    </div>
    <login-form />
  </div>
</template>

<script>
import LoginForm from '@/components/-login-form'

export default {
  components: {
    LoginForm
  }
}
</script>

<style lang="postcss" scoped>
.greeting-block {
  & .greeting {
    @apply mt-5 pt-5 w-auto text-center flex-shrink;

    & h1 {
      @apply text-xl;
    }
    & p {
      @apply text-base my-5 text-left px-4;
    }
  }
}

@screen sm {
  .greeting-block {
    & .greeting {
      @apply px-5;
    }
  }
}
</style>
