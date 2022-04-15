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

    <!-- 对话框 -->
    <el-dialog title="提示" :visible.sync="dialogVisible" width="30%">
      <el-form :model="category">
        <el-form-item label="分类名称">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCategory">确 定</el-button>
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
      // 菜单数组
      menus: [],
      // 需要点击后展开的菜单
      expandedKey: [],
      // menus中数据与el-tree属性对应关系
      defaultProps: {
        children: "children",
        label: "name",
      },
      // 对话框是否可见
      dialogVisible: false,
      // 对话框表单绑定对象
      category: {
        name: "",
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
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
        // console.log(this.menus)
      });
    },

    append(data) {
      // 显示添加对话框
      this.dialogVisible = true;
      // 配置基本属性
      this.category.parentCid = data.catId;
      this.category.catLevel = data.catLevel * 1 + 1;
    },

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

    addCategory() {
      // 发请求
      this.$http({
        url: this.$http.adornUrl("/product/category/save"),
        method: "post",
        data: this.$http.adornData(this.category, false),
      })
        .then((returndata) => {
          // 更新
          this.getMenus();
          //展开父级菜单
          this.expandedKey = [this.category.parentCid];
          // 提示成功
          this.$message({
            type: "success",
            message: "添加成功！",
          });
        })
        .catch(() => {
          // 提示失败
          this.$message.error("添加失败！");
        });
      // 关闭对话框
      this.dialogVisible = false;
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