<!--
针对于 /hotpage 这条路径而显示出来的
在这个组件中, 通过子组件注册的方式, 要显示出Hot.vue这个组件
-->
<template>
  <div class="com-page">
    <hot></hot>
  </div>
</template>

<script>
import HotView from "@/components/HotView.vue";
export default {
  data() {
    return {};
  },
  methods: {},
  components: {
    hot: HotView,
  },
};
</script>

<style lang="less" scoped></style>
