<!--  -->
<template>
  <div>
    <!-- 拖拽开关 -->
    <el-switch
      v-model="draggable"
      active-text="开启拖拽"
      inactive-text="关闭拖拽"
    >
    </el-switch>
    <el-button v-if="draggable" type="primary" @click="batchSave" plain
      >保存拖动</el-button
    >
    <el-button type="danger" @click="batchRemove" plain>批量删除</el-button>

    <!-- 分级菜单展示 -->
    <el-tree
      :data="menus"
      show-checkbox
      node-key="catId"
      :props="defaultProps"
      :default-expanded-keys="expandedKey"
      :expand-on-click-node="false"
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
        <el-form-item label="层级">
          <el-input v-model="category.catLevel" autocomplete="off"></el-input>
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
      // 菜单变更属性
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

      // 拖拽属性
      // 是否允许拖拽
      draggable: false,
      // 总层数
      totalDepth: 3,
      // 被拖拽最大节点深度，默认是1不是0，因为拖拽任意单个元素就已经有1层了
      maxDepth: 1,
      // 需要更新顺序的数组
      updateNodes: [],
      // 更新后需要展开的菜单数量
      updatePcid: [],
    };
  },

  methods: {
    // 刷新表单
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

      // 更新种类信息为待填写状态
      this.category.catId = null;
      this.category.name = null;
      this.category.showStatus = 1;
      this.category.sort = 0;
      this.category.icon = null;
      this.category.productUnit = null;

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
        await this.save("save", this.category, "添加", [
          this.category.parentCid,
        ]);
      }
      if (this.dialogType == "edit") {
        await this.save("update", this.category, "修改", [
          this.category.parentCid,
        ]);
      }
      // 清空并关闭对话框
      this.cleanDialog();
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

    // 请求数据库
    async save(url, data, action, expand) {
      // 发请求
      await this.$http({
        url: this.$http.adornUrl(`/product/category/${url}`),
        method: "post",
        data: this.$http.adornData(data, false),
      })
        .then((data) => {
          // 更新
          this.getMenus();
          //展开父级菜单
          this.expandedKey = expand;
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
    },

    // 删除
    remove(node, data) {
      this.$confirm(`是否删除【${data.name}】菜单`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      }).then(() => {
        // 发请求
        this.save("delete", [data.catId], "删除", [data.parentCid]);
      });
    },

    // 是否允许拖拽
    allowDrop(draggingNode, dropNode, type) {
      // 清空历史数据
      // this.cleanUpdateNodes();

      // 清空最大深度数据，this.updateNodes还需保留，发送请求完再清空
      this.maxDepth = 1;

      // 计算被拖拽节点最大深度
      this.countMaxDepth(draggingNode);

      // 计算被拖拽子树深度
      let dragingTreeDepth = this.maxDepth - draggingNode.level + 1;

      // console.log(
      //   "maxdepth:",
      //   this.maxDepth,
      //   "dragingTreeDepth:",
      //   dragingTreeDepth,
      //   // "draggingNode.data.catLevel:",
      //   // draggingNode.data.catLevel,
      //   "dropNode.level:",
      //   dropNode.level,
      //   // "dropNode.parent.level:",
      //   // dropNode.parent.level,
      //   "this.totalDepth",
      //   this.totalDepth
      // );

      // 计算拖拽完成后是否超过最大子树深度限制
      if (type == "inner") {
        // 与目标节点相加
        return dropNode.level + dragingTreeDepth <= this.totalDepth;
      } else {
        // 与目标节点的父节点相加
        return dropNode.parent.level + dragingTreeDepth <= this.totalDepth;
      }
    },

    // 清空更新列表
    cleanUpdateNodes() {
      this.updateNodes = [];
      this.maxDepth = 1;
      this.updatePcid = [];
    },

    // 计算被拖拽节点最大深度
    countMaxDepth(node) {
      // 如果有子节点，求子节点最大深度，如果没有，就直接拿当前level最为最大深度
      if (node.childNodes != null && node.childNodes.length > 0) {
        for (let i = 0; i < node.childNodes.length; i++) {
          if (node.childNodes[i].level > this.maxDepth) {
            this.maxDepth = node.childNodes[i].level;
          }
          this.countMaxDepth(node.childNodes[i]);
        }
      } else {
        this.maxDepth = node.level;
      }
    },

    // 拖拽成功
    async handleDrop(draggingNode, dropNode, dropType, ev) {
      console.log("tree drop: ", dropNode.label, dropType);

      // 拿到变化列表，及变化后节点信息父节点ID
      let pCid = 0;
      let siblings = null;
      if (dropType != "inner") {
        pCid = dropNode.data.parentCid;
        siblings = dropNode.parent.childNodes;
      } else {
        pCid = dropNode.data.catId;
        siblings = dropNode.childNodes;
      }

      // 更新排列顺序，并生成更新信息
      for (let i = 0; i < siblings.length; i++) {
        // 如果是被拖拽节点
        if (siblings[i].data.catId == draggingNode.data.catId) {
          console.log("变化节点列表", siblings[i]);
          console.log("siblings[i].level", siblings[i].level);
          console.log("draggingNode.level", draggingNode.level);
          console.log("name", siblings[i].data.name);

          let cLevel = draggingNode.level;
          // 且当前节点层级发生改变
          if (siblings[i].level != draggingNode.level) {
            // 修改当前节点层级
            cLevel = siblings[i].level;
            // 修改子节点层级
            this.updateChildNodeLevel(siblings[i]);
            // 保存数据的时候多保存一个层级
          }
          this.updateNodes.push({
            catId: siblings[i].data.catId,
            sort: i,
            parentCid: pCid,
            catLevel: cLevel,
          });
        } else {
          this.updateNodes.push({
            catId: siblings[i].data.catId,
            sort: i,
            parentCid: pCid,
          });
        }
      }

      this.updatePcid.push(pCid);
      // 拖拽成功，离开发送更新请求
      // await this.save("update/sort", this.updateNodes, "菜单顺序", [pCid]);
    },

    // 计算并存储子节点层级
    updateChildNodeLevel(node) {
      if (node.childNodes != null && node.childNodes.length > 0) {
        for (let i = 0; i < node.childNodes.length; i++) {
          this.updateNodes.push({
            catId: node.childNodes[i].data.catId,
            catLevel: node.childNodes[i].level,
          });
          this.updateChildNodeLevel(node.childNodes[i]);
        }
      }
    },

    // 点击保存按钮，批量保存
    async batchSave() {
      console.log("this.updateNodes:", this.updateNodes);
      if (this.updateNodes.length != 0) {
        // 统一发送请求
        await this.save(
          "update/sort",
          this.updateNodes,
          "菜单顺序修改",
          this.updatePcid
        );
      }
      // 清空待更新节点列表
      this.cleanUpdateNodes();
    },

    // 批量删除
    batchRemove() {
      // 过滤tree节点(leafOnly, includeHalfChecked)
      let checkNodes = this.$refs.menuTree.getCheckedNodes();
      let delName = [];
      let delCatId = [];
      for (let i = 0; i < checkNodes.length; i++) {
        delCatId.push(checkNodes[i].catId);
        delName.push(checkNodes[i].name);
      }

      // 发送删除请求
      this.$confirm(`是否删除【${delName}】菜单`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      }).then(() => {
        // 发请求
        this.save("delete", delCatId, "批量删除", []);
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