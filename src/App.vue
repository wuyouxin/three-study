<template>
  <div id="app">
    <demo3 />
  </div>
</template>

<script>
export default {
  name: 'App',
  components: {
    // 转动像机
    // demo1: () => import('./demo1.vue'),
    // demo2: () => import('./demo2.vue')
    demo3: () => import('./demo3.vue')
  }
}
</script>

<style lang="stylus" scoped></style>
