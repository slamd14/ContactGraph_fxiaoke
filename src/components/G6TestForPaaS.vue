<template>
  <div class="test"></div>
</template>

<script>
export default {
  name: "G6TestForPaaS",
  mounted() {
    // 创建一个用于容纳G6绘制的图的容器
    let divNode = document.createElement("div")
    divNode.setAttribute('id', 'mountNode')
    document.querySelector("[class='test']").appendChild(divNode);

    // CDN引入组件并渲染组件
    this.loadScript(this.draw)
  },
  methods: {
    // 加载g6脚本，并调用g6脚本绘图
    loadScript(cb) {
      if(!document.getElementById("g6-min")) {
        let scriptNode = document.createElement("script");
        scriptNode.setAttribute("src", 'https://gw.alipayobjects.com/os/lib/antv/g6/4.3.11/dist/g6.min.js');
        scriptNode.setAttribute("id", 'g6-min');
        scriptNode.onload = function() {
          cb();
        }
        document.getElementsByTagName('body')[0].appendChild(scriptNode);
      } else {
        cb();
      }
    },
    // g6脚本
    draw() {
      const data = {
        // 点集
        nodes: [
          {
            id: 'node1', // String，该节点存在则必须，节点的唯一标识
            x: 100, // Number，可选，节点位置的 x 值
            y: 200, // Number，可选，节点位置的 y 值
          },
          {
            id: 'node2', // String，该节点存在则必须，节点的唯一标识
            x: 300, // Number，可选，节点位置的 x 值
            y: 200, // Number，可选，节点位置的 y 值
          },
        ],
        // 边集
        edges: [
          {
            source: 'node1', // String，必须，起始点 id
            target: 'node2', // String，必须，目标点 id
          },
        ],
      };
      const graph = new G6.Graph({
        container: 'mountNode', // String | HTMLElement，必须，在 Step 1 中创建的容器 id 或容器本身
        width: 800, // Number，必须，图的宽度
        height: 500, // Number，必须，图的高度
      });
      graph.data(data); // 读取 Step 2 中的数据源到图上
      graph.render(); // 渲染图
    }
  }
}
</script>

<style scoped>

</style>