<template>
   
  <div>
    <el-upload
      :action="host"
      :data="dataObj"
      list-type="picture"
      :multiple="false"
      :show-file-list="showFileList"
      :file-list="fileList"
      :before-upload="beforeUpload"
      :on-remove="handleRemove"
      :on-success="handleUploadSuccess"
      :on-preview="handlePreview"
    >
      <el-button size="small" type="primary">点击上传</el-button>
      <div slot="tip" class="el-upload__tip">
        只能上传jpg/png文件，且不超过10MB
      </div>
    </el-upload>
    <!-- 图片展示列表，只有当点击图片预览的时候才会出现 -->
    <el-dialog :visible.sync="dialogVisible">
      <img width="100%" :src="fileList[0].url" alt="" />
    </el-dialog>
  </div>
</template>
<script>
import { policy } from "./policy";
import { getUUID } from "@/utils";

export default {
  name: "singleUpload",
  props: {
    value: String,
  },
  computed: {
    // 计算URL
    imageUrl() {
      return this.value;
    },
    // 计算图片名
    imageName() {
      if (this.value != null && this.value !== "") {
        return this.value.substr(this.value.lastIndexOf("/") + 1);
      } else {
        return null;
      }
    },
    // 根据URL和name形成展示数据
    fileList() {
      return [
        {
          name: this.imageName,
          url: this.imageUrl,
        },
      ];
    },
    // 计算是否展示已上传数据列表
    showFileList: {
      get: function () {
        return (
          this.value !== null && this.value !== "" && this.value !== undefined
        );
      },
      set: function (newValue) {},
    },
  },

  data() {
    return {
      // obs地址
      host: "",
      // 上传数据内容
      dataObj: null,
      //
      dialogVisible: false,
    };
  },
  methods: {
    // 算好的数据扔给父组件
    emitInput(val) {
      this.$emit("input", val);
    },
    // 删除之后把扔出去的组件清空
    handleRemove(file, fileList) {
      this.emitInput("");
    },
    // 图片预览
    handlePreview(file) {
      this.dialogVisible = true;
    },

    // 上传之前组装数据
    beforeUpload(file) {
      let _self = this;
      return new Promise((resolve, reject) => {
        policy()
          .then((response) => {
            _self.dataObj = response.data;
            _self.host = response.data.host;
            _self.$delete(_self.dataObj, "host");
            _self.dataObj.key = _self.dataObj.key + getUUID() + "_${filename}";
            resolve(true);
          })
          .catch((err) => {
            reject(false);
          });
      });
    },

    // 上传成功后组装要扔出去的value
    handleUploadSuccess(res, file) {
      console.log("上传成功...");
      this.showFileList = true;
      this.fileList.pop();
      this.fileList.push({
        name: file.name,
        url:
          this.host + "/" + this.dataObj.key.replace("${filename}", file.name),
      });
      this.emitInput(this.fileList[0].url);
    },
  },
};
</script>
<style>
</style>


