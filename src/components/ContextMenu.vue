<template>
  <div 
  class="container" 
  :style="`left:${position.x}px;top:${position.y}px;`"
  oncontextmenu="return false" 
   >
    <ul>
      <li @click="menuClick('link')">
        <i class="el-icon-share"></i>
        <span>连接</span>
      </li>
      <li @click="menuClick('rename')">
        <i class="el-icon-edit"></i>
        <span>重命名</span>
      </li>
      <li @click="menuClick('delete')">
        <i class="el-icon-delete"></i>
        <span>删除节点</span>
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  name: 'Context-Menu',
  props: ['position'],
  methods: {
    menuClick (option) {
      this.$emit('menuClick', option)
    }
  }
  }
</script>

<style scoped>
ul {
  list-style: none;
  margin: 0;
  padding: 0;
}
.container {
  position: absolute;
  background-color: #FFF;
  box-shadow: 1px 2px 6px 1px rgba(0,0,0,0.2);
}
li {
  cursor: pointer;
  padding: 10px 20px;
  color: #409EFF;
}
li i {
  margin-right: 10px;
}
li:hover {
  background-color: #D9ECFF;
}
</style>
