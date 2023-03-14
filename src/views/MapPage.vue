<!--
针对于 /mappage 这条路径而显示出来的
在这个组件中, 通过子组件注册的方式, 要显示出Map.vue这个组件
-->
<template>
  <div class="com-page">
    <single-map></single-map>
  </div>
</template>

<script>
import MapView from "@/components/MapView.vue";
export default {
  data() {
    return {};
  },
  methods: {},
  components: {
    "single-map": MapView,
  },
};
</script>

<style lang="less" scoped></style>
