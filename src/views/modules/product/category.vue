/* eslint-disable */
<template>
  <div>
    <el-switch
      v-model="draggable"
      active-text="allow dragging"
      inactive-text="stop dragging"
    >
    </el-switch>
    <el-button v-if="draggable" @click="batchSave">BatchSave</el-button>
    <el-button type="danger" @click="batchDelete">Batch Delete</el-button>
    <el-tree
      :data="menus"
      :props="defaultProps"
      @node-click="handleNodeClick"
      :expand-on-click-node="false"
      show-checkbox
      node-key="catId"
      :default-expanded-keys="expandedKey"
      :draggable="draggable"
      :allow-drop="allowDrop"
      @node-drop="handleDrop"
      ref="menuTree"
    >
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            v-if="node.level <= 2"
            type="text"
            size="mini"
            @click="() => append(data)"
          >
            Append
          </el-button>
          <el-button type="text" size="mini" @click="() => edit(data)">
            Edit
          </el-button>
          <el-button
            v-if="node.childNodes.length == 0"
            type="text"
            size="mini"
            @click="() => remove(node, data)"
          >
            Delete
          </el-button>
        </span>
      </span>
    </el-tree>

    <el-dialog
      :title="title"
      :visible.sync="dialogVisible"
      width="30%"
      :before-close="handleClose"
      :close-on-click-modal="false"
    >
      <el-form :model="category">
        <el-form-item label="Category name" :label-width="formLabelWidth">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="Icon" :label-width="formLabelWidth">
          <el-input v-model="category.icon" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="Unit" :label-width="formLabelWidth">
          <el-input
            v-model="category.productUnit"
            autocomplete="off"
          ></el-input>
        </el-form-item>
      </el-form>

      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">Cancel</el-button>
        <el-button type="primary" @click="submitData">Confirm</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
import Vue from "vue";
export default {
  components: {},
  props: {},
  data() {
    return {
      draggable: false,
      pCid: [],
      updateNodes: [],
      maxLevel: 0,
      title: "",
      dialogType: "",
      category: {
        name: "",
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        catId: null,
        productUnit: "",
        icon: "",
      },
      menus: [],
      expandedKey: [],
      dialogVisible: false,
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
        console.log("success getting data...", data.data);
        this.menus = data.data;
      });
    },
    edit(data) {
      console.log("data: ", data);
      this.dialogType = "edit";
      this.title = "Edit a category";
      this.dialogVisible = true;

      // send requests to get the updated data
      this.$http({
        url: this.$http.adornUrl(`/products/category/info/${data.catId}`),
        method: "get",
      }).then(({ data }) => {
        //success
        console.log("data", data);
        this.category.name = data.data.name;
        this.category.catId = data.data.catId;
        this.category.icon = data.data.icon;
        this.category.productUnit = data.data.productUnit;
        this.category.parentCid = data.data.parentCid;
        this.category.catLevel = data.data.catLevel;
        this.category.sort = data.data.sort;
        this.category.showStatus = data.data.showStatus;
      });
    },
    append(data) {
      console.log("append", data);
      this.dialogType = "add";
      this.title = "Add a category";
      this.dialogVisible = true;
      this.category.parentCid = data.catId;
      this.category.catLevel = data.catLevel * 1 + 1;

      this.category.catId = data.catId;
      this.category.icon = "";
      this.category.productUnit = "";
      this.category.sort = 0;
      this.category.showStatus = 1;
      this.category.name = "";
    },
    addCategory() {
      console.log("data: ", this.category);
      this.$http({
        url: this.$http.adornUrl("/products/category/save"),
        method: "post",
        data: this.$http.adornData(this.category, false),
      }).then(({ data }) => {
        this.$message({
          message: "Successfully save the menu!",
          type: "success",
        });
        // close the dialog
        this.dialogVisible = false;
        // refresh new menu
        this.getMenus();
        // set the expanded menu
        this.expandedKey = [this.category.parentCid];
      });
    },
    editCategory() {
      var { catId, name, icon, productUnit } = this.category;

      this.$http({
        url: this.$http.adornUrl("/products/category/update"),
        method: "post",
        data: this.$http.adornData({ catId, name, icon, productUnit }, false),
      }).then(({ data }) => {
        this.$message({
          message: "successful edit",
          type: "success",
        });
        // close the dialog
        this.dialogVisible = false;
        // refresh new menu
        this.getMenus();
        // set the expanded menu
        this.expandedKey = [this.category.parentCid];
      });
    },
    remove(node, data) {
      var ids = [data.catId];
      // pop up warning
      this.$confirm(
        `This will permanently delete ${data.name}. Continue?`,
        "Warning",
        {
          confirmButtonText: "OK",
          cancelButtonText: "Cancel",
          type: "warning",
        }
      )
        .then(() => {
          this.$http({
            url: this.$http.adornUrl("/products/category/delete"),
            method: "post",
            data: this.$http.adornData(ids, false),
          }).then(({ data }) => {
            this.$message({
              message: "Successfully delete the menu!",
              type: "success",
            });
            // refresh the update menu
            this.getMenus();
            // expand the menu
            this.expandedKey = [node.parent.data.catId];
          });
        })
        .catch(() => {});
    },
    allowDrop(draggingNode, dropNode, type) {
      // console.log("allow drop: ", draggingNode, dropNode, type)
      this.maxLevel = 0;
      this.countNodeLevel(draggingNode);
      let depth = Math.abs(this.maxLevel - draggingNode.level) + 1;
      // console.log("maxlevel", this.maxLevel)
      // console.log("depth: ", depth)
      // console.log("dropnode: ", dropNode.level)
      // console.log("dropnode parent: ", dropNode.parent.level)

      if (type == "inner") {
        return depth + dropNode.level <= 3;
      } else {
        return depth + dropNode.parent.level <= 3;
      }
    },
    countNodeLevel(node) {
      //find all the maximum depth
      if (node.childNodes != null && node.childNodes.length > 0) {
        for (let i = 0; i < node.childNodes.length; i++) {
          if (node.childNodes[i].level > this.maxLevel) {
            this.maxLevel = node.childNodes[i].level;
          }
          this.countNodeLevel(node.childNodes[i]);
        }
      }
    },
    handleDrop(draggingNode, dropNode, dropType, ev) {
      console.log("handleDrop: ", draggingNode, dropNode, dropType);
      // get current node's parent id
      let pCid = 0;
      let siblings = null;
      if (dropType == "before" || dropType == "after") {
        pCid =
          dropNode.parent.data.catId == undefined
            ? 0
            : dropNode.parent.childNodes;
        siblings = dropNode.parent.childNodes;
      } else {
        pCid = dropNode.data.catId;
        siblings = dropNode.childNodes;
      }
      this.pCid.push(pCid);
      // get current dragging node's latest order
      for (let i = 0; i < siblings.length; i++) {
        if (siblings[i].data.catId == draggingNode.data.catId) {
          // if we are traversing the current dragging node
          let catLevel = draggingNode.level;
          if (siblings[i].level != draggingNode.level) {
            // current level change
            catLevel = siblings[i].level;
            // child nodes change
            this.updateChildNodeLevel(siblings[i]);
          }
          this.updateNodes.push({
            catId: siblings[i].data.catId,
            sort: i,
            parentCid: pCid,
            catLevel: catLevel,
          });
        } else {
          this.updateNodes.push({ catId: siblings[i].data.catId, sort: i });
        }
      }
      console.log("update nodes", this.updateNodes);
    },
    updateChildNodeLevel(node) {
      if (node.childNodes.length > 0) {
        for (let i = 0; i < node.childNodes.length; i++) {
          var cNode = node.childNodes[i].data;
          this.updateNodes.push({
            catId: cNode.catId,
            catLevel: node.childNodes[i].level,
          });
          this.updateChildNodeLevel(node.childNodes[i]);
        }
      }
    },
    batchSave() {
      this.$http({
        url: this.$http.adornUrl("/products/category/update/sort"),
        method: "post",
        data: this.$http.adornData(this.updateNodes, false),
      }).then(({ data }) => {
        this.$message({
          message: "Successfully changed the menu order!",
          type: "success",
        });

        // refresh the updated menu
        this.getMenus();
        // expand the menu
        this.expandedKey = this.pCid;
        this.updateNodes = [];
        this.maxLevel = 0;
      });
    },
    batchDelete() {
      let catIds = [];
      let checkedNodes = this.$refs.menuTree.getCheckedNodes();
      console.log("checked nodes", checkedNodes);
      for (let i = 0; i < checkedNodes.length; i++) {
        catIds.push(checkedNodes[i].catId);
      }
      this.$confirm(
        `This will permanently delete ${catIds}. Continue?`,
        "Warning",
        {
          confirmButtonText: "OK",
          cancelButtonText: "Cancel",
          type: "warning",
        }
      )
        .then(() => {
          this.$http({
            url: this.$http.adornUrl("/products/category/delete"),
            method: "post",
            data: this.$http.adornData(catIds, false),
          }).then(({ data }) => {
            this.$message({
              message: "Batch delete finished!",
              type: "success"
            })
            this.getMenus();
          });
        })
        .catch(() => {});
    },
    handleNodeClick() {},
    submitData(data) {
      if (this.dialogType == "add") {
        this.addCategory();
      }
      if (this.dialogType == "edit") {
        this.editCategory();
      }
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