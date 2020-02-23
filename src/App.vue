<template>
  <div class="container">
    hello {{state.message}}
    <div>数字: {{count}}</div>
    <div>两倍数字: {{doubleCount}}</div>
    <my-button name="增加" @get="increment"></my-button>
  </div>
</template>

<script>
import { computed, ref, reactive } from 'vue'
import MyButton from './components/MyButton.vue'
export default {
  components: {
    MyButton
  },
  setup () {
    const count = ref(0)

    const state = reactive({
      message: 'Vue3'
    })

    const doubleCount = computed(() => count.value * 2)

    const increment = e => {
      console.log(e)
      count.value += 10
      state.message = 'Vue3. Good!'
    }

    return {
      count,
      state,
      doubleCount,
      increment
    }
  }
}
</script>
<style lang="scss">
body {
  font-size: 16px;
}
.container {
  text-align: center;
  line-height: 2;
}
</style>
