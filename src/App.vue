<template>
  <div id="app">
    <demo1 />
  </div>
</template>

<script>
export default {
  name: 'App',
  components: {
    // 转动像机
    demo1: () => import('./demo1.vue')
  }
}
</script>

<style lang="stylus" scoped></style>
