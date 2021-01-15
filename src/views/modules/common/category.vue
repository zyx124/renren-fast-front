<template>
  <el-tree
    :data="menus"
    :props="defaultProps"
    node-key="catId"
    ref="menuTree"
    @node-click="nodeClick"
  ></el-tree>
</template>
<script>
export default {
  components: {},
  props: {},
  data() {
    return {
      menus: [],
      expandedKey: [],
      defaultProps: {
        children: "children",
        label: "name",
      },
    };
  },
  computed: {},
  watch: {},
  methods: {
    getMenus() {
      this.$http({
        url: this.$http.adornUrl("/products/category/list/tree"),
        method: "get",
      }).then(({ data }) => {
        // console.log("success getting data...", data.data);
        this.menus = data.data;
      });
    },
    nodeClick(data, node, component) {
      console.log("After Node click: ",data, node, component);

      // send parent component events
      this.$emit("tree-node-click", data, node, component);
    },
  },
  created() {
    this.getMenus();
  },
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