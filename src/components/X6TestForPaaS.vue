<template>
  <div class="test">
    <div class="describe">联系人之间的关系线代表介绍关系</div>
    <div class="controlScreen">
      <fx-button type="primary" round @click="toFullScreen">全屏显示</fx-button>
    </div>
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

      curObjId: '', // 当前(客户)对象id
      curObjName: '', // 当前(客户)对象name

      isFullScreen: false, // 是否全屏
    }
  },
  mounted() {
    // 获取客户对象id
    this.fetchData()

    // 创建一个用于容纳X6绘制的图的容器
    let divNode = document.createElement('div')
    divNode.setAttribute('id', 'container')
    divNode.setAttribute('style', 'margin: 0 auto')
    document.querySelector("[class='test']").appendChild(divNode)

    let parameters = [{
      type: 'string',
      name: 'objId',
      value: this.curObjId
    }]
    // 获取联系人对象列表数据
    FxUI.userDefine.call_controller('getContactInGraph__c', parameters).then((res) => {
      if (res.Result.StatusCode === 0) {
        this.rawObjData = res.Value.functionResult.dataList
        console.log('联系人对象列表数据为: ', this.rawObjData)
        // CDN引入antv-x6组件
        this.loadScript(this.draw)
      }
    })
  },
  methods: {
    // 全屏展示
    toFullScreen() {
      document.getElementById('container').requestFullscreen()
      this.isFullScreen = true
    },
    // 如果该组件配置在对象详情页，那么获取当前对象id
    fetchData() {
      this.curObjId = this.$context.getData()['_id']
      this.curObjName = this.$context.getData()['name']
    },

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
      g.setGraph({ rankdir: that.dir, nodesep: 200, ranksep: 250 }) // nodesep: 节点之间横向连线长度 rankesep: 节点之间纵向距离长度
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
            width: 400,
            height: 180,
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
                  class: 'department',
                },
              },
              {
                tagName: 'text',
                attrs: {
                  class: 'name',
                },
              },
              {
                tagName: 'text',
                attrs: {
                  class: 'job_title'
                }
              },
              {
                tagName: 'text',
                attrs: {
                  class: 'relationship'
                }
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
                fill: '#fd7f00',
                stroke: '#fd7f00',
                strokeWidth: 1,
                pointerEvents: 'visiblePainted',
                cursor: 'pointer'
              },
              '.image': {
                x: 16,
                y: 40,
                width: 100,
                height: 100,
                opacity: 0.7,
                cursor: 'pointer'
              },
              '.department': {
                refX: 0.95,
                refY: 0.3,
                fill: '#fff',
                fontFamily: 'Arial',
                fontSize: 26,
                textAnchor: 'end',
                textVerticalAnchor: 'middle',
                cursor: 'pointer'
              },
              '.name': {
                refX: 0.95,
                refY: 0.55,
                fill: '#fff',
                fontFamily: 'Arial',
                fontSize: 20,
                fontWeight: '300',
                textAnchor: 'end',
                cursor: 'pointer'
              },
              '.job_title': {
                refX: 0.95,
                refY: 0.7,
                fill: '#fff',
                fontFamily: 'Arial',
                fontSize: 20,
                fontWeight: '300',
                textAnchor: 'end',
                cursor: 'pointer'
              },
              '.relationship': {
                refX: 0.95,
                refY: 0.85,
                fill: '#fff',
                fontFamily: 'Arial',
                fontSize: 20,
                fontWeight: '300',
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
              contactId: '', // 联系人id
              gender: '', // 联系人性别
              contactAllData: {} // 该联系人所有数据, 方便日后拓展
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
        console.log('当前点击的联系人的所有数据为: ', cell['data']['contactAllData'])
        // 如果是全屏状态则关闭全屏
        if (this.isFullScreen) {
          document.exitFullscreen()
          this.isFullScreen = false
        }
        FxUI.objectUIAction.viewObject('ContactObj', cell['data']['contactId'])
      })
    },
    // 监听点击加号
    addChild() {
      let that = this
      this.graph.on('node:add', ({ e, node }) => {
        e.stopPropagation()

        // TODO 创建新节点
        let nodeData = node['data']['contactAllData']
        // 1. PaaS平台弹出新建联系人对象弹框
        FxUI.objectUIAction.addObject('ContactObj', {
          objectData: {
            'field_Jl19V__c': true,
            "introducer": nodeData['_id'],
            "introducer__r": nodeData['name'], // 查找关联字段不仅要填id， name也必须要填写
            "account_id": that.curObjId,
            "account_id__r": that.curObjName, // 查找关联字段不仅要填id， name也必须要填写
          },
          recordType: 'default__c',
          showType: 'full'
        }).then(res => {
          let newContact = res['Value']['objectData']
          // 2. 获取新建结果
          // 3. UI创建新节点
          const member = that.createNode(newContact['department'], newContact['name'], newContact['_id'], newContact['gender'], newContact)
          that.graph.freeze()

          // 新节点作为原节点的子节点, 创建边
          that.graph.addCell([member, that.createEdge(node, member)])
          that.layout()
        }).catch(err => {
          // handle error
        })
      })
    },
    // 创建节点
    async createNode(department, name, contactId, gender, contactAllData) {
      let that = this
      let resultNode = {}
      // 名片npath
      let cardNpath = contactAllData['card'].length === 0 ? '' : contactAllData['card'][0]['path']
      if (cardNpath === '') {
        resultNode = this.graph.createNode({
          shape: 'org-node',
          attrs: {
            '.image': { xlinkHref: gender === '1' ? that.male : that.female },
            '.department': {
              // text: that.Dom.breakText('部门: ' + department, { width: 160, height: 45 }),
              text: that.Dom.breakText('公司: ' + department, { width: 160, height: 45 }),
            },
            '.name': {
              text: that.Dom.breakText('姓名: ' + name, { width: 160, height: 45 }),
            },
            '.job_title': {
              text: that.Dom.breakText('职位: ' + contactAllData['job_title'], { width: 160, height: 45 }),
            },
            '.relationship': {
              text: that.Dom.breakText('关系: ' + contactAllData['field_oRBqR__c'], { width: 160, height: 45 }),
            },
          },
          data: {
            'contactId': contactId,
            'gender': gender === '1' ? '男' : '女',
            'contactAllData': contactAllData
          }
        })
      } else {
        // 调用自定义控制器获取一个名片临时url
        let parameters = [{type: 'string', name: 'fileNpath', value: cardNpath}]
        await FxUI.userDefine.call_controller('getUrlByNpath__c', parameters).then((res) => {
          let that2 = that
          if (res.Result.StatusCode === 0) {
            let fileUrl = res['Value']['functionResult'][0]['url']
            console.log('附件url', fileUrl)
            resultNode = that2.graph.createNode({
              shape: 'org-node',
              attrs: {
                '.image': { xlinkHref: fileUrl },
                '.department': {
                  // text: that2.Dom.breakText('部门: ' + department, { width: 160, height: 45 }),
                  text: that2.Dom.breakText('公司: ' + department, { width: 160, height: 45 }),
                },
                '.name': {
                  text: that2.Dom.breakText('姓名: ' + name, { width: 160, height: 45 }),
                },
                '.job_title': {
                  text: that2.Dom.breakText('职位: ' + contactAllData['job_title'], { width: 160, height: 45 }),
                },
                '.relationship': {
                  text: that2.Dom.breakText('关系: ' + contactAllData['field_oRBqR__c'], { width: 160, height: 45 }),
                },
              },
              data: {
                'contactId': contactId,
                'gender': gender === '1' ? '男' : '女',
                'contactAllData': contactAllData
              }
            })
          }
        })
      }
      return resultNode
    },
    // 创建边
    createEdge(source, target) {
      return this.graph.createEdge({
        shape: 'org-edge',
        source: { cell: source.id },
        target: { cell: target.id },
        // router: { // TODO 边的路由
        //   name: 'manhattan',
        //   args: {
        //     startDirections: ['top'],
        //     endDirections: ['bottom'],
        //   }
        // },
        // router: {
        //   name: 'metro',
        //   args: {
        //     startDirections: ['top'],
        //     endDirections: ['bottom'],
        //   },
        // },
        attrs: {
          line: { // 箭头
            targetMarker: 'block',
            strokeWidth: 5,
            stroke: 'black'
          },
        },
        labels: [
          {
            attrs: {
              label: {
                text: '介绍',
                fontSize: 30
              }
            },
            position: 0.5
          },
        ]
      })
    },

    // createEdge2(source, target) { // TODO 测试其他关系线
    //   return this.graph.createEdge({
    //     shape: 'org-edge',
    //     source: { cell: source.id },
    //     target: { cell: target.id },
    //     labels: [
    //       {
    //         attrs: {
    //           label: {
    //             text: '其他关系其他关系',
    //             stroke: 'red'
    //           },
    //         },
    //         position: 0.5
    //       },
    //     ],
    //     attrs: {
    //       line: {
    //         strokeWidth: 5,
    //         stroke: '#d500dc',
    //       },
    //     },
    //   })
    // },

    // 返回id为introducerId的节点
    getNodeByIntroducerId(introducerId) {
      for (let node of this.nodes) {
        let curContactId = node['data']['contactId']
        if (curContactId === introducerId)
          return node
      }
      return null
    },
    // 绘图
    async draw() {
      // 注册节点样式
      this.registerNode()
      // 注册边样式
      this.registerEdge()
      // 初始化画布对象
      this.graph = new this.Graph({
        container: document.getElementById('container'),
        width: 1150,
        height: 850,
        background: {
          color: '#fdfdfd'
        },
        interacting: false,
        panning: true, // 画布平移
        mousewheel: { // 画布缩放
          enabled: true,
          modifiers: ['ctrl', 'meta'],
        },
      })

      for (let contact of this.rawObjData) {
        let node = await this.createNode(contact['department'] == null ? '' : contact['department'], contact['name'], contact['_id'], contact['gender'], contact) // 调用自定义控制器发送了网络请求，所以createNode为一个异步方法, 应该用await阻塞在这里
        console.log('当前节点为: ', node)
        this.nodes.push(node)
      }
      console.log('当前各个节点为: ', this.nodes)

      // 依据介绍人构建节点上下级关系
      for (let node of this.nodes) {
        let curIntroducer = (node['data']['contactAllData']['introducer'] == null) ? '' : node['data']['contactAllData']['introducer']
        if (curIntroducer === '') {
          // 无介绍人
          continue
        }
        let parentNode = this.getNodeByIntroducerId(curIntroducer)
        if (parentNode != null) { // 有介绍人，但是该介绍人没有关联当前客户
          this.edges.push(this.createEdge(parentNode, node))
        }
      }

      // TODO 测试其他关系
      // this.edges.push((this.createEdge2(this.nodes[11], this.nodes[12]))) // TODO 测试其他关系 小马与小猪
      // this.edges.push((this.createEdge2(this.nodes[1], this.nodes[7]))) // TODO 测试其他关系 小黑与小红
      // this.edges.push((this.createEdge2(this.nodes[7], this.nodes[6]))) // TODO 测试其他关系 小红与小绿
      // this.edges.push((this.createEdge2(this.nodes[10], this.nodes[8]))) // TODO 测试其他关系 小羊和小明

      // this.edges.push((this.createEdge(this.nodes[0], this.nodes[1]))) // TODO 测试 3和2
      // this.edges.push((this.createEdge(this.nodes[0], this.nodes[2]))) // TODO 测试 3和1
      this.graph.resetCells([...this.nodes, ...this.edges])
      this.layout()
      this.graph.zoomTo(0.4)
      this.graph.centerContent()
      this.setup()
    }
  }
}
</script>

<style scoped>
.controlScreen {
  padding-top: 10px;
  font-size: 20px;
  margin-left: 10px;
}
.describe {
  padding-top: 20px;
  margin: 0 10px 10px 10px;
  font-size: 20px;
}
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