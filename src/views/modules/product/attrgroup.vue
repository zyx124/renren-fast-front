<template>
  <el-row :gutter="20">
    <el-col :span="6"><category>cate</category></el-col>
    <el-col :span="18"><div class="grid-content bg-purple">Form</div></el-col>
  </el-row>
</template>
<script>
import Category from "../common/category";

export default {
  components: { Category },
  props: {},
  data() {
    return {};
  },
  computed: {},
  watch: {},
  methods: {},
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