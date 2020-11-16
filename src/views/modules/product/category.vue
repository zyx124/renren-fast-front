<template>
  <el-tree :data="data" :props="defaultProps" @node-click="handleNodeClick"></el-tree>
</template>
<script>
export default {
  components: {},
  props: {},
  data() {},
 
  computed: {},
  watch: {},
  methods: {
      handleNodeClick(data) {
        console.log(data);
      }
    },
  created() {},
  mounted() {},
  beforeCreate() {},
  beforeMount() {},
  beforeUpdate() {},
  updated() {},
  beforeDestroy() {},
  destroyed() {},
  activated() {},
};
</script>
<style scoped></style>