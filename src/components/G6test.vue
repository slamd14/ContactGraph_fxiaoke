<template>
  <div>
    <div id="mountNode">

    </div>
  </div>
</template>

<script>
import graphJS from './graph-20230726141530'
export default {
  mounted() {
    graphJS.init()
  }
}
</script>

<style scoped>
#mountNode {
  text-align: center;
}
</style>

<!-- 这里的style不能加scoped属性，因为标签是通过js脚本引入的，并不属于组件-->
<style>
/* 提示框的样式 */
.g6-tooltip {
  border: 1px solid #e2e2e2;
  border-radius: 4px;
  font-size: 12px;
  color: #545454;
  background-color: rgba(255, 255, 255, 0.9);
  padding: 10px 8px;
  box-shadow: rgb(174, 174, 174) 0px 0px 10px;
}
</style>