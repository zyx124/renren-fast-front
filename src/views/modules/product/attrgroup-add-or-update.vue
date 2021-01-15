<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible"
    @closed="dialogClose"
  >
    <el-form
      :model="dataForm"
      :rules="dataRule"
      ref="dataForm"
      @keyup.enter.native="dataFormSubmit()"
      label-width="80px"
    >
      <el-form-item label="Group Name" prop="attrGroupName">
        <el-input v-model="dataForm.attrGroupName" placeholder=""></el-input>
      </el-form-item>
      <el-form-item label="Sort" prop="sort">
        <el-input v-model="dataForm.sort" placeholder=""></el-input>
      </el-form-item>
      <el-form-item label="Description" prop="descript">
        <el-input v-model="dataForm.descript" placeholder=""></el-input>
      </el-form-item>
      <el-form-item label="Icon" prop="icon">
        <el-input v-model="dataForm.icon" placeholder=""></el-input>
      </el-form-item>
      <el-form-item label="CatalogID" prop="catelogId">
        <!-- <el-input v-model="dataForm.catelogId" placeholder=""></el-input> -->
        <el-cascader
          v-model="dataForm.catelogPath"
          :options="categories"
          :props="props"
          filterable
          placeholder="Type to search"
        ></el-cascader>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
export default {
  data() {
    return {
      props: {
        value: "catId",
        label: "name",
        children: "children"
      },
      visible: false,
      dataForm: {
        categories: [],
        attrGroupId: 0,
        attrGroupName: "",
        sort: "",
        descript: "",
        icon: "",
        catelogPath: [],
        catelogId: 0
      },
      dataRule: {
        attrGroupName: [
          { required: true, message: "不能为空", trigger: "blur" },
        ],
        sort: [{ required: true, message: "不能为空", trigger: "blur" }],
        descript: [{ required: true, message: "不能为空", trigger: "blur" }],
        icon: [{ required: true, message: "不能为空", trigger: "blur" }],
        catelogId: [{ required: true, message: "不能为空", trigger: "blur" }],
      },
    };
  },
  methods: {
    dialogClose() {
      this.dataForm.catelogPath = []
    },
    getCategories() {
      this.$http({
        url: this.$http.adornUrl("/products/category/list/tree"),
        method: "get",
      }).then(({ data }) => {
        // console.log("success getting data...", data.data);
        this.categories = data.data;
      });
    },
    init(id) {
      this.dataForm.attrGroupId = id || 0;
      this.visible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].resetFields();
        if (this.dataForm.attrGroupId) {
          this.$http({
            url: this.$http.adornUrl(
              `/products/attrgroup/info/${this.dataForm.attrGroupId}`
            ),
            method: "get",
            params: this.$http.adornParams(),
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.dataForm.attrGroupName = data.attrGroup.attrGroupName;
              this.dataForm.sort = data.attrGroup.sort;
              this.dataForm.descript = data.attrGroup.descript;
              this.dataForm.icon = data.attrGroup.icon;
              this.dataForm.catelogId = data.attrGroup.catelogId;
              // get the full path of the catelog
              this.dataForm.catelogPath = data.attrGroup.catelogPath;
            }
          });
        }
      });
    },
    // 表单提交
    dataFormSubmit() {
      this.$refs["dataForm"].validate((valid) => {
        if (valid) {
          this.$http({
            url: this.$http.adornUrl(
              `/products/attrgroup/${
                !this.dataForm.attrGroupId ? "save" : "update"
              }`
            ),
            method: "post",
            data: this.$http.adornData({
              attrGroupId: this.dataForm.attrGroupId || undefined,
              attrGroupName: this.dataForm.attrGroupName,
              sort: this.dataForm.sort,
              descript: this.dataForm.descript,
              icon: this.dataForm.icon,
              catelogId: this.dataForm.catelogPath[this.dataForm.catelogPath.length - 1],
            }),
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.$message({
                message: "操作成功",
                type: "success",
                duration: 1500,
                onClose: () => {
                  this.visible = false;
                  this.$emit("refreshDataList");
                },
              });
            } else {
              this.$message.error(data.msg);
            }
          });
        }
      });
    },
  },
  created() {
    this.getCategories()
  }
};
</script>
