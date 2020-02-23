<template>
  <button class="btn" @click="onClick">{{props.name}}</button>
</template>

<script>
import { defineComponent } from "vue";

export default {
  setup (props, {emit}) {
    const onClick = () => {
      console.log('clicked')
      emit('get', Date.now())
    }

    return {
      onClick,
      props
    }
  } 
}
</script>

<style lang="scss">
.btn {
  font-weight: bold;
}
button {
  border: 0;
  font-size: 18px;
  color: #fff;
  background: #1375e6;
  &:focus {
    outline: none;
  }
}
</style>