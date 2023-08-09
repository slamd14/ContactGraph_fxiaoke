<template>
  <div class="test">
    <!-- 页面加载进度条 -->
    <div class="loading_bg">
      <div class="jiazai_div">
        <i></i>
        <i></i>
        <i></i>
        <i></i>
        <i></i>
      </div>
    </div>
  </div>
</template>

<script>
/**
 * TODO:
 * 1. 自定义控制器获取 联系人 对象数据
 */
export default {
  name: "X6TestForPaaS",
  data() {
    return {
      Graph: {}, // 画布类
      Cell: {},
      Node: {},
      Color: {},
      Dom: {},
      graph: {}, // 画布对象

      nodes: [],
      edges: [],

      male: 'https://gw.alipayobjects.com/mdn/rms_43231b/afts/img/A*kUy8SrEDp6YAAAAAAAAAAAAAARQnAQ',
      female: 'https://gw.alipayobjects.com/mdn/rms_43231b/afts/img/A*f6hhT75YjkIAAAAAAAAAAAAAARQnAQ',
      dir: 'TB', // 布局方向 LR RL TB BT

      rawObjData: {}, // 初始线性数据
    }
  },
  mounted() {
    // 创建一个用于容纳X6绘制的图的容器
    let divNode = document.createElement('div')
    divNode.setAttribute('id', 'container')
    divNode.setAttribute('style', 'margin: 0 auto')
    document.querySelector("[class='test']").appendChild(divNode)

    // 获取联系人对象列表数据
    FxUI.userDefine.call_controller('getContactInGraph__c').then((res) => {
      if (res.Result.StatusCode === 0) {
        this.rawObjData = res.Value.functionResult.dataList
        console.log('联系人对象列表数据为: ', this.rawObjData)
        // CDN引入antv-x6组件
        this.loadScript(this.draw)
      }
    })
  },
  // beforeDestroy() {
  //   location.reload()
  // },
  methods: {
    /**
     * 令加载进度条消失
     * @param el: 进度条dom元素
     */
    fadeOut(el) {
      el.style.opacity = 1;
      (function fade() {
        if ((el.style.opacity -= .1) < 0) {
          el.style.display = "none";
        } else {
          requestAnimationFrame(fade);
        }
      })();
    },
    // 加载x6脚本
    loadScript(cb) {
      if (!document.getElementById("d3")) {
        let scriptNode = document.createElement("script");
        scriptNode.setAttribute("src", 'https://d3js.org/d3.v3.min.js');
        scriptNode.setAttribute("id", 'd3');
        scriptNode.setAttribute("charset", 'utf-8');
        document.getElementsByTagName('body')[0].appendChild(scriptNode);
        let that = this
        scriptNode.onload = function () {
          if (!document.getElementById("dagre")) {
            let scriptNode1 = document.createElement("script");
            scriptNode1.setAttribute("src", 'https://cdn.bootcss.com/dagre-d3/0.4.18/dagre-d3.js');
            scriptNode1.setAttribute("id", 'dagre');
            document.getElementsByTagName('body')[0].appendChild(scriptNode1);
            let that1 = that
            scriptNode1.onload = function () {
              if(!document.getElementById("x6")) {
                let scriptNode2 = document.createElement("script");
                scriptNode2.setAttribute("src", 'https://unpkg.com/@antv/x6@1.31.0/dist/x6.js');
                scriptNode2.setAttribute("id", 'x6');
                let that2 = that1
                scriptNode2.onload = function() {
                  console.log('CDN引入外部脚本d3、dagre、antv-x6完成')
                  that.Graph = X6.Graph
                  that.Cell = X6.Cell
                  that.Node = X6.Node
                  that.Color = X6.Color
                  that.Dom = X6.Dom
                  cb();
                  that2.fadeOut(document.querySelector("[class = 'loading_bg']")) // 等脚本完全引入后再令进度条消失
                }
                document.getElementsByTagName('body')[0].appendChild(scriptNode2);
              }
            }
          }
        }
      } else {
        console.log('CDN引入外部脚本d3、dagre、antv-x6完成')
        this.Graph = X6.Graph
        this.Cell = X6.Cell
        this.Node = X6.Node
        this.Color = X6.Color
        this.Dom = X6.Dom
        cb();
        this.fadeOut(document.querySelector("[class = 'loading_bg']"))
      }
    },
    // 自动布局
    layout() {
      const nodes = this.graph.getNodes()
      const edges = this.graph.getEdges()
      const g = new dagreD3.dagre.graphlib.Graph()
      let that = this
      g.setGraph({ rankdir: that.dir, nodesep: 50, ranksep: 50 })
      g.setDefaultEdgeLabel(() => ({}))

      const width = 260
      const height = 90
      nodes.forEach((node) => {
        g.setNode(node.id, { width, height })
      })

      edges.forEach((edge) => {
        const source = edge.getSource()
        const target = edge.getTarget()
        g.setEdge(source.cell, target.cell)
      })

      dagreD3.dagre.layout(g)
      this.graph.freeze()

      g.nodes().forEach((id) => {
        const node = this.graph.getCell(id)
        if (node) {
          const pos = g.node(id)
          node.position(pos.x, pos.y)
        }
      })

      edges.forEach((edge) => {
        const source = edge.getSourceNode()
        const target = edge.getTargetNode()
        const sourceBBox = source.getBBox()
        const targetBBox = target.getBBox()

        // console.log(sourceBBox, targetBBox)

        if ((that.dir === 'LR' || that.dir === 'RL') && sourceBBox.y !== targetBBox.y) {
          const gap =
              that.dir === 'LR'
                  ? targetBBox.x - sourceBBox.x - sourceBBox.width
                  : -sourceBBox.x + targetBBox.x + targetBBox.width
          const fix = that.dir === 'LR' ? sourceBBox.width : 0
          const x = sourceBBox.x + fix + gap / 2
          edge.setVertices([
            { x, y: sourceBBox.center.y },
            { x, y: targetBBox.center.y },
          ])
        } else if (
            (that.dir === 'TB' || that.dir === 'BT') &&
            sourceBBox.x !== targetBBox.x
        ) {
          const gap =
              that.dir === 'TB'
                  ? targetBBox.y - sourceBBox.y - sourceBBox.height
                  : -sourceBBox.y + targetBBox.y + targetBBox.height
          const fix = that.dir === 'TB' ? sourceBBox.height : 0
          const y = sourceBBox.y + fix + gap / 2
          edge.setVertices([
            { x: sourceBBox.center.x, y },
            { x: targetBBox.center.x, y },
          ])
        } else {
          edge.setVertices([])
        }
      })

      this.graph.unfreeze()
    },
    // 监听自定义事件
    setup() {
      this.addChild()
      this.mouseEnterListener()
    },
    // 注册节点样式
    registerNode() {
      this.Graph.registerNode(
          'org-node',
          {
            width: 260,
            height: 88,
            markup: [
              {
                tagName: 'rect',
                attrs: {
                  class: 'card',
                },
              },
              {
                tagName: 'image',
                attrs: {
                  class: 'image',
                },
              },
              {
                tagName: 'text',
                attrs: {
                  class: 'rank',
                },
              },
              {
                tagName: 'text',
                attrs: {
                  class: 'name',
                },
              },
              {
                tagName: 'g',
                attrs: {
                  class: 'btn add',
                },
                children: [
                  {
                    tagName: 'circle',
                    attrs: {
                      class: 'add',
                    },
                  },
                  {
                    tagName: 'text',
                    attrs: {
                      class: 'add',
                    },
                  },
                ],
              },
            ],
            attrs: {
              '.card': {
                rx: 10,
                ry: 10,
                refWidth: '100%',
                refHeight: '100%',
                fill: '#5F95FF',
                stroke: '#5F95FF',
                strokeWidth: 1,
                pointerEvents: 'visiblePainted',
                cursor: 'pointer'
              },
              '.image': {
                x: 16,
                y: 16,
                width: 56,
                height: 56,
                opacity: 0.7,
                cursor: 'pointer'
              },
              '.rank': {
                refX: 0.95,
                refY: 0.5,
                fill: '#fff',
                fontFamily: 'Courier New',
                fontSize: 13,
                textAnchor: 'end',
                textVerticalAnchor: 'middle',
                cursor: 'pointer'
              },
              '.name': {
                refX: 0.95,
                refY: 0.7,
                fill: '#fff',
                fontFamily: 'Arial',
                fontSize: 14,
                fontWeight: '600',
                textAnchor: 'end',
                cursor: 'pointer'
              },
              '.btn.add': {
                refDx: -16,
                refY: 16,
                event: 'node:add',
                cursor: 'pointer'
              },
              '.btn > circle': {
                r: 10,
                fill: 'transparent',
                stroke: '#fff',
                strokeWidth: 1,
              },
              '.btn.add > text': {
                fontSize: 20,
                fontWeight: 800,
                fill: '#fff',
                x: -5.5,
                y: 7,
                fontFamily: 'Times New Roman',
                text: '+',
              },
            },
            data: {
              contactId: '' // 联系人id
            }
          },
          true,
      )
    },
    // 注册边样式
    registerEdge() {
      this.Graph.registerEdge(
          'org-edge',
          {
            zIndex: -1,
            attrs: {
              line: {
                strokeWidth: 2,
                stroke: '#A2B1C3',
                sourceMarker: null,
                targetMarker: null,
              },
            },
          },
          true,
      )
    },
    // 监听鼠标点击节点
    mouseEnterListener() {
      this.graph.on('node:click', ({ e, x, y, cell, view }) => {
        // console.log('当前点击的联系人的id为: ', cell['data']['contactId'])
        FxUI.objectUIAction.viewObject('ContactObj', cell['data']['contactId'])
      })
    },
    // 监听点击加号
    addChild() {
      let that = this
      this.graph.on('node:add', ({ e, node }) => {
        e.stopPropagation()
        const member = that.createNode(
            'Employee',
            'New Employee',
            Math.random() < 0.5 ? this.male : this.female,
        )
        that.graph.freeze()
        that.graph.addCell([member, that.createEdge(node, member)])
        that.layout()
        console.log('点击加号')
      })
    },
    createNode(rank, name, image, contactId) {
      let that = this
      return this.graph.createNode({
        shape: 'org-node',
        attrs: {
          '.image': { xlinkHref: image },
          '.rank': {
            text: that.Dom.breakText(rank, { width: 160, height: 45 }),
          },
          '.name': {
            text: that.Dom.breakText(name, { width: 160, height: 45 }),
          },
        },
        data: {
          'contactId': contactId
        }
      })
    },
    createEdge(source, target) {
      return this.graph.createEdge({
        shape: 'org-edge',
        source: { cell: source.id },
        target: { cell: target.id },
      })
    },
    // 绘图
    draw() {
      // 注册节点样式
      this.registerNode()
      // 注册边样式
      this.registerEdge()
      // 初始化画布对象
      this.graph = new this.Graph({
        container: document.getElementById('container'),
        width: 1600,
        height: 800,
        background: {
          color: '#F2F7FA'
        },
        interacting: false,
        panning: true, // 画布平移
        mousewheel: { // 画布缩放
          enabled: true,
          modifiers: ['ctrl', 'meta'],
        },
      })

      // TODO 对联系人对象列表是数据进行树状化 solveData
      // TODO 构建联系人节点
      // TODO 构建联系人关系
      for (let contact of this.rawObjData) {
        this.nodes.push(this.createNode('层级x', contact['name'], this.male, contact['_id']))
      }

      this.edges.push(this.createEdge(this.nodes[0], this.nodes[1]))
      this.edges.push(this.createEdge(this.nodes[1], this.nodes[2]))
      this.edges.push(this.createEdge(this.nodes[1], this.nodes[3]))
      this.edges.push(this.createEdge(this.nodes[1], this.nodes[4]))
      this.edges.push(this.createEdge(this.nodes[1], this.nodes[5]))

      // const nodes = [
      //   this.createNode('Founder & Chairman', 'Pierre Omidyar', this.male),
      //   this.createNode('President & CEO', 'Margaret C. Whitman', this.female),
      //   this.createNode('President, PayPal', 'Scott Thompson', this.male),
      //   this.createNode('President, Ebay Global Marketplaces', 'Devin Wenig', this.male),
      //   this.createNode('Senior Vice President Human Resources', 'Jeffrey S. Skoll', this.male),
      //   this.createNode('Senior Vice President Controller', 'Steven P. Westly', this.male),
      // ]

      // const edges = [
      //   this.createEdge(nodes[0], nodes[1]),
      //   this.createEdge(nodes[1], nodes[2]),
      //   this.createEdge(nodes[1], nodes[3]),
      //   this.createEdge(nodes[1], nodes[4]),
      //   this.createEdge(nodes[1], nodes[5]),
      // ]
      // 画布显示
      // this.graph.resetCells([...nodes, ...edges])
      this.graph.resetCells([...this.nodes, ...this.edges])
      this.layout()
      this.graph.zoomTo(0.8)
      this.graph.centerContent()
      this.setup()
    }
  }
}
</script>

<style scoped>
/* 加载等待 */

/*.loading_bg {*/
/*  background: #fff;*/
/*  position: fixed;*/
/*  z-index: 1000;*/
/*  width: 100%;*/
/*  height: 100%;*/
/*}*/

.loading_bg {
  background: #fff;
  display: table-cell;
  vertical-align: middle;
  text-align: center;
}

.jiazai_div {
  position: absolute;
  margin: auto;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  width: 40px;
  height: 40px;
}

.jiazai_div i {
  float: left;
  display: block;
  width: 4px;
  height: 40px;
  margin: 0 2px;
  background: orangered;
  -webkit-animation: load 1.2s infinite;
  animation: load 1.2s infinite;
  -webkit-transform: scaleY(0.4);
  -ms-transform: scaleY(0.4);
  transform: scaleY(0.4);
}

.jiazai_div i:nth-child(2) {
  -webkit-animation-delay: 0.1s;
  animation-delay: 0.2s;
}

.jiazai_div i:nth-child(3) {
  -webkit-animation-delay: 0.2s;
  animation-delay: 0.3s;
}

.jiazai_div i:nth-child(4) {
  -webkit-animation-delay: 0.3s;
  animation-delay: 0.4s;
}

.jiazai_div i:nth-child(5) {
  -webkit-animation-delay: 0.4s;
  animation-delay: 0.5s;
}

@-webkit-keyframes load {

  0%, 40%, 100% {
    -webkit-transform: scaleY(0.4);
    transform: scaleY(0.4);
  }
  20% {
    -webkit-transform: scaleY(1);
    transform: scaleY(1);
  }
}

@keyframes load {
  0%, 40%, 100% {
    -webkit-transform: scaleY(0.4);
    transform: scaleY(0.4);
  }
  20% {
    -webkit-transform: scaleY(1);
    transform: scaleY(1);
  }
}

.loading {
  height: 100%;
  width: 100%;
}
</style>