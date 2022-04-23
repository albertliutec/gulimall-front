<template>
  <el-dialog
    :title="!dataForm.attrGroupId ? '新增' : '修改'"
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
      <el-form-item label="组名" prop="attrGroupName">
        <el-input
          v-model="dataForm.attrGroupName"
          placeholder="组名"
        ></el-input>
      </el-form-item>
      <el-form-item label="排序" prop="sort">
        <el-input v-model="dataForm.sort" placeholder="排序"></el-input>
      </el-form-item>
      <el-form-item label="描述" prop="descript">
        <el-input v-model="dataForm.descript" placeholder="描述"></el-input>
      </el-form-item>
      <el-form-item label="组图标" prop="icon">
        <el-input v-model="dataForm.icon" placeholder="组图标"></el-input>
      </el-form-item>
      <el-form-item label="所属分类id" prop="catelogId">
        <!-- <el-input v-model="dataForm.catelogId" placeholder="所属分类id"></el-input> -->
        <el-cascader
          v-model="dataForm.catelogPath"
          :options="categorys"
          :props="props"
          placeholder="搜索"
          filterable
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
      // 级联选择器显示
      props: {
        value: "catId",
        label: "name",
        children: "children",
      },
      visible: false,
      categorys: [],

      // 级联选择器修改或删除部分
      dataForm: {
        attrGroupId: 0,
        attrGroupName: "",
        sort: "",
        descript: "",
        icon: "",
        catelogId: "",
        // 同时用于显示，新增，删除
        catelogPath: [],
      },

      // 前端数据校验
      // 修改或删除校验
      dataRule: {
        attrGroupName: [
          { required: true, message: "组名不能为空", trigger: "blur" },
        ],
        sort: [{ required: true, message: "排序不能为空", trigger: "blur" }],
        descript: [
          { required: true, message: "描述不能为空", trigger: "blur" },
        ],
        icon: [{ required: true, message: "组图标不能为空", trigger: "blur" }],
        // catelogId: [
        //   { required: true, message: "所属分类id不能为空", trigger: "blur" },
        // ],
        // 显示校验
        catelogPath: [
          { required: true, message: "所属分类id不能为空", trigger: "blur" },
        ],
      },
    };
  },
  methods: {
    // 获取category菜单
    getCategorys() {
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get",
      }).then(({ data }) => {
        this.categorys = data.entityList;
      });
    },

    init(id) {
      this.dataForm.attrGroupId = id || 0;
      this.visible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].resetFields();
        // 当修改或者删除的时候，id不是0才会调用此方法
        if (this.dataForm.attrGroupId) {
          this.$http({
            url: this.$http.adornUrl(
              `/product/attrgroup/info/${this.dataForm.attrGroupId}`
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
              // 查出catId的完整路径
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
              `/product/attrgroup/${
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
              // 取最后一级菜单才是三级分类菜单
              catelogId:
                this.dataForm.catelogPath[this.dataForm.catelogPath.length - 1],
            }),
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.$message({
                message: "操作成功",
                type: "success",
                duration: 1500,
                onClose: () => {
                  this.visible = false;
                  // 通知父组件更新分类表
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

    // 取消提交
    dialogClose() {
      this.dataForm.catelogPath = [];
    },
  },

  created() {
    this.getCategorys();
  },
};
</script>
