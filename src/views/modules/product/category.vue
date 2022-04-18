<!--  -->
<template>
  <div>
    <!-- 分级菜单展示 -->
    <el-tree
      :data="menus"
      show-checkbox
      node-key="catId"
      :props="defaultProps"
      :default-expanded-keys="expandedKey"
      :expand-on-click-node="false"
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
            edit
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

    <!-- 添加，修改对话框 -->
    <el-dialog
      :close-on-click-modal="false"
      :title="dialogTitle"
      :visible.sync="dialogVisible"
      width="30%"
    >
      <el-form :model="category">
        <el-form-item label="分类名称">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="图标">
          <el-input v-model="category.icon" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="计量单位">
          <el-input
            v-model="category.productUnit"
            autocomplete="off"
          ></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="cleanDialog">取 消</el-button>
        <el-button type="primary" @click="submitData">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>


<script>
//这里可以导入其他文件（比如：组件，工具js，第三方插件js，json文件，图片文件等等）
//例如：import 《组件名称》 from '《组件路径》';

export default {
  //import引入的组件需要注入到对象中才能使用
  components: {},

  data() {
    return {
      // 菜单属性
      // 菜单数组
      menus: [],
      // 需要点击后展开的菜单
      expandedKey: [],
      // menus中数据与el-tree属性对应关系
      defaultProps: {
        children: "children",
        label: "name",
      },

      // 对话框属性
      // 对话框tittle
      dialogTitle: null,
      // 对话框是否可见
      dialogVisible: false,
      // 对话框类型
      dialogType: null,

      // 对话框表单绑定对象
      category: {
        catId: null,
        name: null,
        parentCid: null,
        catLevel: null,
        showStatus: 1,
        sort: 0,
        icon: null,
        productUnit: null,
      },
    };
  },

  methods: {
    getMenus() {
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get",
      }).then(({ data }) => {
        this.menus = data.entityList;
      });
    },

    // 打开添加对话框
    async append(data) {
      this.dialogTitle = "添加分类";
      this.dialogType = "append";
      await this.getBaseInfo(data);
      // 获取上级菜单ID
      this.category.parentCid = this.category.catId;
      this.category.catLevel = this.category.catLevel * 1 + 1;
      console.log(this.category);

      // 更新种类信息为待填写状态
      this.category.catId = null;
      this.category.name = null;
      this.category.showStatus = 1;
      this.category.sort = 0;
      this.category.icon = null;
      this.category.productUnit = null;
      // console.log(this.category);

      // 显示添加对话框
      this.dialogVisible = true;
    },

    // 打开修改对话框
    async edit(data) {
      this.dialogTitle = "修改分类";
      this.dialogType = "edit";
      await this.getBaseInfo(data);
      // console.log(this.category);
      // 显示添加对话框
      this.dialogVisible = true;
    },

    // 提交数据
    async submitData() {
      if (this.dialogType == "append") {
        await this.save("save", "添加");
      }
      if (this.dialogType == "edit") {
        await this.save("update", "修改");
      }
    },

    // 获取某种类全量信息
    async getBaseInfo(data) {
      await this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: "get",
      }).then(({ data }) => {
        // 配置基本属性
        this.category = data.category;
      });
    },

    // 清空并关闭对话框
    cleanDialog() {
      // 属性清空
      this.category.catId = null;
      this.category.name = null;
      this.category.parentCid = null;
      this.category.catLevel = null;
      this.category.showStatus = 1;
      this.category.sort = 0;
      this.category.icon = null;
      this.category.productUnit = null;
      // 对话框清空
      this.dialogTitle = null;
      this.dialogVisible = false;
    },

    // 添加保存数据库
    async save(url, action) {
      // 发请求
      await this.$http({
        url: this.$http.adornUrl(`/product/category/${url}`),
        method: "post",
        data: this.$http.adornData(this.category, false),
      })
        .then((data) => {
          // 更新
          this.getMenus();
          //展开父级菜单
          this.expandedKey = [this.category.parentCid];
          // 提示成功
          this.$message({
            type: "success",
            message: `${action}成功！`,
          });
        })
        .catch(() => {
          // 提示失败
          this.$message.error(`${action}失败！`);
        });
      // 清空并关闭对话框
      this.cleanDialog();
    },

    // 删除
    remove(node, data) {
      this.$confirm(`是否删除【${data.name}】菜单`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      }).then(() => {
        // 发请求
        this.$http({
          url: this.$http.adornUrl("/product/category/delete"),
          method: "post",
          data: this.$http.adornData([data.catId], false),
        })
          .then((returndata) => {
            // 更新
            this.getMenus();
            //展开父级菜单
            this.expandedKey = [data.parentCid];
            // 提示成功
            this.$message({
              type: "success",
              message: "删除成功！",
            });
          })
          .catch(() => {
            // 提示失败
            this.$message.error("删除失败！");
          });
      });
    },
  },
  //监听属性 类似于data概念
  computed: {},
  //监控data中的数据变化
  watch: {},
  //生命周期 - 创建完成（可以访问当前this实例）
  created() {},
  //生命周期 - 挂载完成（可以访问DOM元素）
  mounted() {
    this.getMenus();
  },
  beforeCreate() {}, //生命周期 - 创建之前
  beforeMount() {}, //生命周期 - 挂载之前
  beforeUpdate() {}, //生命周期 - 更新之前
  updated() {}, //生命周期 - 更新之后
  beforeDestroy() {}, //生命周期 - 销毁之前
  destroyed() {}, //生命周期 - 销毁完成
  activated() {}, //如果页面有keep-alive缓存功能，这个函数会触发
};
</script>
<style scoped>
</style>