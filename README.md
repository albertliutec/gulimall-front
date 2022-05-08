# gulimall-front
## 项目介绍
gulimall-front基于[renren-fast-vue](https://gitee.com/renrenio/renren-fast-vue?_from=gitee_search)添加功能开发，基于vue、element-ui构建开发环境，实现电商平台后台管理系统的前端部分功能。后端项目可见[gulimall-backend](https://github.com/albertliutec/gulimall-backend)

## 项目改动
相较于[尚硅谷课程内容](https://www.bilibili.com/video/BV1np4y1C7Yf?spm_id_from=333.337.search-card.all.click)前端部分做了如下改动：
- src\views\modules\product\category.vue 修改了三级菜单不能拖动bug，并做了深度封装（修复了后台递归组装树结构内存溢出问题）
- 图片上传采用华为云OBS存储图片
- PubSub传值丢失问题
- attrupdate页面路由丢失问题
- spinadd页面 attr显示格式丢失的问题
- 其他细小bug

## 项目课程
请查看：https://www.bilibili.com/video/BV1np4y1C7Yf?spm_id_from=333.337.search-card.all.click

