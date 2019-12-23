<template>
  <el-container style="height: 100vh;">
    <!-- 页眉 -->
    <el-header class="header-container">
      <h1 class="title">简易拓扑图</h1>
    </el-header>
    <el-container style="height: 100%">
      <!-- 侧边节点库 -->
      <el-aside class="aside">
        <!-- 侧栏菜单 -->
        <el-menu>
          <!-- 子菜单 -->
          <el-submenu v-for="(type, index) in typeList" :key="type" :index="String(index)">
            <template slot="title">
              <i class="el-icon-s-grid"></i>
              <span>{{type}}</span>
            </template>
            <el-menu-item-group>
              <el-menu-item
              v-for="item in libraryList[type]"
              :key="item.id"
              class="library-item">
                <div draggable="true" @dragstart="dragToBoardStart">
                  <img :src="item.pic" :alt="item.name" draggable="false">
                  <span>{{item.name}}</span>
                </div>
              </el-menu-item>
            </el-menu-item-group>
          </el-submenu>
        </el-menu>
      </el-aside>
      <!-- 拓扑画板 -->
      <el-main class="board-container">
        <!-- topo中的节点的右键菜单 -->
        <Context-Menu :position="position" v-if="showMenu" @menuClick="clickMenuItem" />
        <!-- 画板 -->
        <svg 
        class="board" 
        ondragover="return false" 
        @drop="dropToBoard" 
        oncontextmenu="return false"
        @click.left.stop.prevent="closeContextMenu">
          <!-- 已连接的线 -->
          <line
          v-for="(item, index) in lines"
          :key="index"
          :x1="item.x1" :y1="item.y1"
          :x2="item.x2" :y2="item.y2"
          style="stroke:rgb(255,0,0);stroke-width:2"/>
          <!-- 正在连接的线 -->
          <line
          :x1="connecting.x1" :y1="connecting.y1"
          :x2="connecting.x2" :y2="connecting.y2"
          style="stroke:rgb(255,0,0);stroke-width:2"/>
          <!-- topo图上的节点 -->
          <g 
          v-for="(item, index) in nodeList"
          :key="item.id"
          @mousedown.left.stop.prevent="moveAndLink(index, $event)"
          @click.right.stop.prevent="nodeMenu(index, $event)"
          >
            <image :xlink:href="item.pic" width="50px" height="50px" :x="item.x" :y="item.y"></image>
            <text :x="item.x + 25" :y="item.y + 66" style="text-anchor: middle; user-select: none;">{{item.name}}</text>
          </g>
        </svg>
      </el-main>
    </el-container>
    <el-footer height="60px" style="border-top: 1px solid #CCC;">
      <a href="https://github.com/laddwong" class="address">项目地址</a>
    </el-footer>
  </el-container>
</template>

<script>
import ContextMenu from './ContextMenu'
import { MessageBox } from 'element-ui'
import nodeData from '../data/nodeData'
export default {
  name: 'Topo',
  components: {
    ContextMenu
  },
  data () {
    return {
      libraryList: {}, // 左侧节点库的节点数据
      typeList: [], // 节点分类
      nodeList: [], // 在topo图中的节点数据
      connecting:{ // 显示正在连接的线条
        x1: 0,
        y1: 0,
        x2: 0,
        y2: 0
      },
      move: true, // 操作模式，默认为移动。可切换为连接模式
      position: {x: 0, y: 0}, // 右键菜单的位置
      showMenu: false, // 控制右键菜单的显示与否
      indexOfMenu: null // 表示在哪个节点上点击了右键菜单
    }
  },
  methods: {
    // 从左边的节点库拖出节点
    dragToBoardStart (e) {
      // 设置拖出的数据
      e.dataTransfer.setData('text/plain', JSON.stringify({pic: e.target.children[0].src, name: e.target.children[1].innerText}))
      e.dataTransfer.effectAllowed = "copy" // 设置拖的操作为复制操作
      // window.console.log(e)
    },
    // 节点拖放到topo图区域，即新建节点
    dropToBoard (e) {
      const content = JSON.parse(e.dataTransfer.getData('text/plain')) // 接收来自拖出的内容,并还原为对象
      let node = {
        id: Symbol(), // 唯一id
        pic: content.pic, // 图片
        name: content.name, // 默认显示名称，可修改
        x: e.layerX, // 横坐标
        y: e.layerY, // 纵坐标
        link: [] // 连接
      }
      this.nodeList.push(node)
    },
    // 移动topo图中的节点，连接节点
    moveAndLink (index, e) {
      // 判断当前模式
      if (this.move) {
        // 移动模式
        const layerX = e.layerX - this.nodeList[index].x
        const layerY = e.layerY - this.nodeList[index].y
        document.onmousemove = (e) => {
          this.nodeList[index].x = e.layerX - layerX 
          this.nodeList[index].y = e.layerY - layerY
        }
        document.onmouseup = () => {
          document.onmousemove = null
          document.onmouseup = null
        }
      } else {
        // 连线模式
        this.nodeList[this.indexOfMenu].link.push(this.nodeList[index].id) // 在节点中增加连线终点的id
        this.connecting = { // 重置正在连接的线
          x1: 0,
          y1: 0,
          x2: 0,
          y2: 0
        }
        document.onmousemove = null // 重置鼠标移动事件
        this.move = true // 重置为移动模式
      }
    },
    // 显示topo图上的节点的右键菜单
    nodeMenu (index, e) {
      this.position = {x: e.clientX, y: e.clientY}
      this.showMenu = true
      this.indexOfMenu = index
    },
    // 执行右键菜单的功能
    clickMenuItem (option) {
      // 关闭右键菜单
      this.showMenu = false
      // 连接功能
      if (option === 'link') {
        // 设置为连线模式
        this.move = false
        // 创建连线
        this.connecting = {
          x1: this.nodeList[this.indexOfMenu].x + 20,
          y1: this.nodeList[this.indexOfMenu].y + 20,
          x2: this.nodeList[this.indexOfMenu].x + 20,
          y2: this.nodeList[this.indexOfMenu].y + 20
        }
        // 连线终点跟随鼠标
        document.onmousemove = (e) => {
          this.connecting.x2 = e.layerX
          this.connecting.y2 = e.layerY
        }
      }
      // 重命名功能
      if (option === 'rename') {
        MessageBox.prompt('请输入新名称', '重命名', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          inputPattern: /\S/,
          inputErrorMessage: '名称不能为空'
        }).then(({ value }) => {
          this.nodeList[this.indexOfMenu].name = value
        })
      }
      // 删除功能
      if (option === 'delete') {
        MessageBox.confirm(`是否删除节点 "${this.nodeList[this.indexOfMenu].name}"`, '删除节点', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.nodeList.splice(this.indexOfMenu, 1)
        })
      }
    },
    // 点击空白地方，关闭右键菜单，取消连线模式
    closeContextMenu () {
      // 关闭右键菜单
      this.position = {x: 0, y: 0}
      this.showMenu = false
      this.indexOfMenu = null
      // 取消连线模式
      this.move = true
      this.connecting = {
        x1: 0,
        y1: 0,
        x2: 0,
        y2: 0
      }
      document.onmousemove = null
    }
  },
  computed: {
    // 节点之间的连线
    lines () {
      let tempLines = []
      const idList = this.nodeList.map(item => { return item.id })
      // 遍历所有节点
      this.nodeList.forEach(item => {
        // 连线起点的坐标
        let startX = item.x
        let startY = item.y
        // 遍历终点
        item.link.forEach(destination => {
          let destinationIndex = idList.indexOf(destination)
          if (destinationIndex !== -1) {
            // 终点存在，建立连线
            tempLines.push({
              x1: startX + 20,
              y1: startY + 20,
              x2: this.nodeList[destinationIndex].x + 20,
              y2: this.nodeList[destinationIndex].y + 20
            })
          }
        })
      })
      return tempLines
    }
  },
  mounted () {
    this.libraryList = nodeData
    this.typeList = Object.keys(this.libraryList)
  }
}
</script>

<style scoped>

.aside {
  background-color: #F2F6FC;
  border-right: 2px solid #F2F6FC;
  height: 100%;
}
.library-item {
  color: #606266;
}
.library-item img {
  width: 30px;
  height: 30px;
  margin-right: 10px;
}
.board-container {
  padding: 0;
}
.board {
  background-color: #FFF;
  height: 99%;
  width: 100%;
}
.header-container {
  background-color: #409EFF;
}
.title {
  margin: 0;
  line-height: 60px;
  color: #FFF;
}
.address {
  color: #409EFF;
  line-height: 60px;
}
</style>
