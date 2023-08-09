<template>
  <div class="test" ref="test">
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
<!--     悬浮框 -->
    <div v-if="isShowNodeTipsPanel" :style="{left: nodeMenuPanelPosition.x + 'px', top: nodeMenuPanelPosition.y + 'px' }" style="z-index: 999;padding:10px;background-color: #ffffff;border:#eeeeee solid 1px;box-shadow: 0px 0px 8px #cccccc;position: absolute;">
      <div style="line-height: 25px;color: #888888;font-size: 12px; text-align: center">节点名称：{{currentNode.text}}</div>
      <div class="c-node-menu-item">id:{{currentNode.text}}</div>
    </div>
  </div>
</template>

<script>
export default {
  name: "GraphTestForPaaS",
  data() {
    return {
      isShowNodeTipsPanel: false,
      nodeMenuPanelPosition: { x: 0, y: 0 },
      currentNode: {},
    }
  },
  mounted() {
    // 创建挂载组件的dom标签
    let divNode = document.createElement('div')
    divNode.setAttribute('id', 'myapp')
    document.querySelector("[class = 'test']").appendChild(divNode)

    let divNode2 = document.createElement('div')
    divNode2.setAttribute('style', 'width: calc(100% - 10px);height:calc(100vh - 140px);')
    divNode2.setAttribute('v-loading', 'g_loading')
    divNode.appendChild(divNode2)

    let relationGraph = document.createElement('relation-graph')
    relationGraph.setAttribute('ref', 'seeksRelationGraph')
    relationGraph.setAttribute(':options', 'graphOptions')
    relationGraph.setAttribute(':on-node-click', 'onNodeClick')
    relationGraph.setAttribute(':on-line-click', 'onLineClick')
    relationGraph.setAttribute(':on-node-expand', 'onNodeExpand')
    relationGraph.innerHTML = '<div slot="node" slot-scope="{node}" @mouseover="showNodeTips(node, $event)" @mouseout="hideNodeTips(node, $event)" style="cursor: pointer; display: flex; justify-content: center; align-items: center; height: 100%">\n' +
        '          <div style="color: white;font-size: 16px;padding-left: 5px;padding-right: 5px;">\n' +
        '            {{ node.text }}\n' +
        '          </div>\n' +
        '        </div>' // TODO hover触发悬浮框
    divNode2.appendChild(relationGraph)

    // CDN引入组件并渲染组件
    this.loadScript(this.initForRelationGraph)
  },
  beforeDestroy() {
    // 1.组件销毁前清理掉引入的vue脚本，防止与PaaS平台的vue2版本冲突
    let myVue2 = document.getElementById('myVue2')
    myVue2.remove()
    // 2.从文档树(sources中)移除vue@2.6.13，这里暂时不知道如何删除指定文件，就直接给了个页面刷新 // TODO 四个js脚本本地化, 利用vue-cli运行时(到底如何填写静态文件路径, 每次都在前面自动加了个localhost:8080)，<script>的src属性直接引入，记得改relation-graph.min.js里的依赖html2canvas与screenfull的源码，加上./
    location.reload()
  },
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

    /**
     * 动态添加script标签引入第三方js脚本
     * @param cb: 第三方js脚本加载完成后动作
     */
    loadScript(cb) {
      // vue2.6.13, 版本低了的话不适配relation-graph
      if(!document.getElementById('myVue2')) {
        let scriptNode = document.createElement('script')
        scriptNode.setAttribute('src', 'https://cdn.jsdelivr.net/npm/vue@2.6.13')
        scriptNode.setAttribute('id', 'myVue2')
        document.getElementsByTagName('body')[0].appendChild(scriptNode);
      }
      // screenfull组件
      if(!document.getElementById("my-screenfull")) {
        let scriptNode = document.createElement("script");
        scriptNode.setAttribute("src", 'https://cdnjs.cloudflare.com/ajax/libs/screenfull.js/5.1.0/screenfull.min.js');
        scriptNode.setAttribute("id", 'my-screenfull');
        document.getElementsByTagName('body')[0].appendChild(scriptNode);
      }
      // html2canvas组件
      if(!document.getElementById("my-html2canvas")) {
        let scriptNode = document.createElement("script");
        scriptNode.setAttribute("src", 'https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js');
        scriptNode.setAttribute("id", 'my-html2canvas');
        document.getElementsByTagName('body')[0].appendChild(scriptNode);
      }
      // relation-graph组件
      if(!document.getElementById("my-graph")) {
        let scriptNode = document.createElement("script");
        scriptNode.setAttribute("src", 'https://unpkg.com/relation-graph/vue2/relation-graph.umd.js');
        scriptNode.setAttribute("id", 'my-graph');
        let that = this
        scriptNode.onload = function () {
          console.log('RelationGraph脚本加载完毕')
          console.log('脚本为: ', RelationGraph)
          cb(RelationGraph)
          that.fadeOut(document.querySelector("[class = 'loading_bg']")) // 等脚本完全引入后再令加载进度条消失
        }
        document.getElementsByTagName('body')[0].appendChild(scriptNode);
      } else {
        this.fadeOut(document.querySelector("[class = 'loading_bg']")) // 脚本已存在，直接令进度条消失
      }
    },

    /**
     * RelationGraph组件初始化
     * @param e: RelationGraph脚本暴露出的对象
     */
    initForRelationGraph(e) {
      const RelationGraph = Vue.extend(e)
      let that = this
      let graph = new RelationGraph ({
        el: '#myapp',
        data: {
          g_loading: true,
          graphOptions: {
            'backgrounImage': 'https://camo.githubusercontent.com/ede1654f055903cdc39044129d15d5b158f4f3b33ba5b7c21c7407792a506dea/687474703a2f2f72656c6174696f6e2d67726170682e636f6d2f776562736974652f6c6f676f',
            'backgrounImageNoRepeat': true,
            'layouts': [
              {
                'label': '中心',
                'layoutName': 'tree',
                'layoutClassName': 'seeks-layout-tree',
                'defaultJunctionPoint': 'border',
                'defaultNodeShape': 0,
                'defaultLineShape': 1,
                // 'centerOffset_x': -50,
                // 'centerOffset_y': 0,
                'min_per_width': '60',
                'min_per_height': '400'
              }
            ],
            'defaultExpandHolderPosition': 'bottom',
            'defaultLineShape': 4,
            'defaultJunctionPoint': 'tb',
            'defaultNodeShape': 1,
            'defaultNodeWidth': '50',
            'defaultNodeHeight': '250',
            'defaultNodeBorderWidth': 0
          },
        },
        mounted() {
          this.setGraphData();
        },
        methods: {
          setGraphData() {
            const __graph_json_data = { 'rootId': 'N1', 'nodes': [{ 'id': 'N1', 'text': '深圳市腾讯计算机系统有限公司', 'color': '#2E4E8F' }, { 'id': 'N2', 'text': '张志东', 'color': '#4ea2f0' }, { 'id': 'N3', 'text': '陈一丹', 'color': '#4ea2f0' }, { 'id': 'N4', 'text': '许晨晔', 'color': '#4ea2f0' }, { 'id': 'N5', 'text': '马化腾', 'color': '#4ea2f0' }, { 'id': 'N6', 'text': '腾讯云科技有限公司', 'color': '#4ea2f0' }, { 'id': 'N7', 'text': '腾讯医疗健康（深圳）有限公司', 'color': '#4ea2f0' }, { 'id': 'N8', 'text': '深圳市腾讯视频文化传播有限公司', 'color': '#4ea2f0' }, { 'id': 'N9', 'text': '星创互联（北京）科技有限公司', 'color': '#4ea2f0' }, { 'id': 'N10', 'text': '苏州钟鼎创业二号投资中心（有限合伙）', 'color': '#4ea2f0' }, { 'id': 'N11', 'text': '北京驿码神通信息技术有限公司', 'color': '#4ea2f0' }, { 'id': 'N12', 'text': '张家界（北京驿码神通）信息技术有限公司', 'color': '#4ea2f0' }, { 'id': 'N13', 'text': '滨海（天津）金融资产交易中心股份有限公司', 'color': '#4ea2f0' }, { 'id': 'N14', 'text': '深圳腾富博投资有限公司', 'color': '#4ea2f0' }, { 'id': 'N15', 'text': '腾讯影业文化传播有限公司', 'color': '#4ea2f0' }, { 'id': 'N16', 'text': '霍尔果斯晓腾影业文化传播有限公司', 'color': '#4ea2f0' }, { 'id': 'N17', 'text': '苍穹互娱（天津）文化传播有限公司', 'color': '#4ea2f0' }, { 'id': 'N18', 'text': '北京腾讯影业有限公司', 'color': '#4ea2f0' }, { 'id': 'N19', 'text': '霍尔果斯腾影影视发行有限公司', 'color': '#4ea2f0' }, { 'id': 'N20', 'text': '上海腾闻网络科技有限公司', 'color': '#4ea2f0' }, { 'id': 'N21', 'text': '上海宝申数字科技有限公司', 'color': '#4ea2f0' }, { 'id': 'N22', 'text': '海南高灯科技有限公司', 'color': '#4ea2f0' }, { 'id': 'N23', 'text': '益盟股份有限公司', 'color': '#4ea2f0' }, { 'id': 'N24', 'text': '北京魔方无限科技有限公司', 'color': '#4ea2f0' }, { 'id': 'N25', 'text': '北京像素软件科技股份有限公司', 'color': '#4ea2f0' }, { 'id': 'N26', 'text': '深圳市世纪腾华信息技术有限公司', 'color': '#4ea2f0' }, { 'id': 'N27', 'text': '浙江齐聚科技有限公司', 'color': '#4ea2f0' }, { 'id': 'N28', 'text': '未来电视有限公司', 'color': '#4ea2f0' }, { 'id': 'N29', 'text': '北京腾新科技有限公司', 'color': '#4ea2f0' }, { 'id': 'N30', 'text': '河北雄安新区腾讯计算机系统有限公司', 'color': '#4ea2f0' }, { 'id': 'N31', 'text': '深圳市企鹅金融科技有限公司', 'color': '#4ea2f0' }, { 'id': 'N32', 'text': '深圳市移卡科技有限公司', 'color': '#4ea2f0' }, { 'id': 'N33', 'text': '财付通支付科技有限公司', 'color': '#4ea2f0' }, { 'id': 'N34', 'text': '金保信社保卡科技有限公司', 'color': '#4ea2f0' }, { 'id': 'N35', 'text': '网联清算有限公司', 'color': '#4ea2f0' }, { 'id': 'N36', 'text': '北京搜狗信息服务有限公司', 'color': '#4ea2f0' }, { 'id': 'N37', 'text': '北京网罗天下生活科技有限公司', 'color': '#4ea2f0' }, { 'id': 'N120', 'text': '深圳市腾讯商业管理有限公司', 'color': '#4ea2f0' }, { 'id': 'N121', 'text': '深圳市智税链科技有限公司', 'color': '#4ea2f0' }, { 'id': 'N122', 'text': '横琴红土创新创业投资合伙企业（有限合伙）', 'color': '#4ea2f0' }, { 'id': 'N123', 'text': '上海挚信新经济一期股权投资合伙企业（有限合伙）', 'color': '#4ea2f0' }, { 'id': 'N124', 'text': '上海云锋股权投资中心（有限合伙）', 'color': '#4ea2f0' }, { 'id': 'N125', 'text': '北京创新工场投资中心（有限合伙）', 'color': '#4ea2f0' }, { 'id': 'N126', 'text': '广州市擎天柱网络科技有限公司', 'color': '#4ea2f0' }, { 'id': 'N127', 'text': '河南腾河网络科技有限公司', 'color': '#4ea2f0' }, { 'id': 'N128', 'text': '深圳市财付通网络金融小额贷款有限公司', 'color': '#4ea2f0' }, { 'id': 'N129', 'text': '湖北腾楚网络科技有限责任公司', 'color': '#4ea2f0' }, { 'id': 'N130', 'text': '腾讯征信有限公司', 'color': '#4ea2f0' }, { 'id': 'N131', 'text': '百行征信有限公司', 'color': '#4ea2f0' }, { 'id': 'N132', 'text': '广东腾南网络信息科技有限公司', 'color': '#4ea2f0' }, { 'id': 'N133', 'text': '深圳市腾南网络信息科技有限公司', 'color': '#4ea2f0' }, { 'id': 'N134', 'text': '广州腾威会展有限公司', 'color': '#4ea2f0' }, { 'id': 'N135', 'text': '广州南极广告传媒有限公司', 'color': '#4ea2f0' }, { 'id': 'N136', 'text': '广州壹糖网络科技有限公司', 'color': '#4ea2f0' }, { 'id': 'N137', 'text': '广州玩心艺网络科技有限公司', 'color': '#4ea2f0' }, { 'id': 'N138', 'text': '广东腾南网络信息科技有限公司深圳分公司', 'color': '#4ea2f0' }, { 'id': 'N139', 'text': '珠海横琴腾南网络信息科技有限公司', 'color': '#4ea2f0' }, { 'id': 'N140', 'text': '这个节点原本是没有子节点的', 'color': 'rgba(255, 120, 0, 1)', 'fontColor': '#000000' }, { 'id': 'N141', 'text': '上海腾讯企鹅影视文化传播有限公司', 'color': '#4ea2f0' }, { 'id': 'N142', 'text': '海南周天娱乐有限公司', 'color': '#4ea2f0' }, { 'id': 'N143', 'text': '杭州红杉合远股权投资合伙企业（有限合伙）', 'color': '#4ea2f0' }, { 'id': 'N144', 'text': '广州银汉科技有限公司', 'color': '#4ea2f0' }, { 'id': 'N145', 'text': '深圳市文娱华彩科技有限公司', 'color': '#4ea2f0' }, { 'id': 'N146', 'text': '林芝文娱本源科技有限公司', 'color': '#4ea2f0' }, { 'id': 'N147', 'text': '深圳市文娱华章科技有限公司', 'color': '#4ea2f0' }, { 'id': 'N148', 'text': '腾讯大地通途（北京）科技有限公司', 'color': '#4ea2f0' }, { 'id': 'N149', 'text': '苏州钟鼎三号创业投资中心（有限合伙）', 'color': '#4ea2f0' }, { 'id': 'N150', 'text': '永杨安风（北京）科技股份有限公司', 'color': '#4ea2f0' }, { 'id': 'N151', 'text': '霍尔果斯永杨安风网络科技有限公司', 'color': '#4ea2f0' }, { 'id': 'N152', 'text': '辽宁腾辽科技有限公司', 'color': '#4ea2f0' }, { 'id': 'N153', 'text': '沈阳小黄牛网络科技有限公司', 'color': '#4ea2f0' }, { 'id': 'N154', 'text': '深圳市泰捷软件技术有限公司', 'color': '#4ea2f0' }, { 'id': 'N155', 'text': '众安在线财产保险股份有限公司', 'color': '#4ea2f0' }, { 'id': 'N156', 'text': '深圳市腾讯动漫有限公司', 'color': '#4ea2f0' }, { 'id': 'N157', 'text': '北京奇迹开元文化有限公司', 'color': '#4ea2f0' }, { 'id': 'N158', 'text': '浙江腾讯影业有限公司', 'color': '#4ea2f0' }, { 'id': 'N159', 'text': '北京醋溜网络科技股份有限公司', 'color': '#4ea2f0' }, { 'id': 'N160', 'text': '甘肃刚泰控股（集团）股份有限公司', 'color': '#4ea2f0' }, { 'id': 'N161', 'text': '浙江腾越网络科技有限公司', 'color': '#4ea2f0' }, { 'id': 'N162', 'text': '杭州热秀网络技术有限公司', 'color': '#4ea2f0' }, { 'id': 'N163', 'text': '浙江腾趣网络科技有限公司', 'color': '#4ea2f0' }, { 'id': 'N164', 'text': '湖南腾湘科技有限公司', 'color': '#4ea2f0' }, { 'id': 'N165', 'text': '湖南绘装网络科技有限公司', 'color': '#4ea2f0' }, { 'id': 'N166', 'text': '华谊兄弟传媒股份有限公司', 'color': '#4ea2f0' }, { 'id': 'N167', 'text': '无锡买卖宝信息技术有限公司', 'color': '#4ea2f0' }, { 'id': 'N168', 'text': '优扬文化传媒股份有限公司', 'color': '#4ea2f0' }, { 'id': 'N169', 'text': '武汉鲨鱼网络直播技术有限公司', 'color': '#4ea2f0' }, { 'id': 'N170', 'text': '深圳市腾讯网域计算机网络有限公司', 'color': '#4ea2f0' }, { 'id': 'N171', 'text': '厦门国际金融技术有限公司', 'color': '#4ea2f0' }, { 'id': 'N172', 'text': '深圳市移卡科技有限公司', 'color': '#4ea2f0' }, { 'id': 'N173', 'text': '上海企鹅金融信息服务有限公司', 'color': '#4ea2f0' }, { 'id': 'N174', 'text': '腾安基金销售（深圳）有限公司', 'color': '#4ea2f0' }, { 'id': 'N175', 'text': '深圳微众金融科技集团股份有限公司', 'color': '#4ea2f0' }, { 'id': 'N176', 'text': '深圳瓶子科技有限公司', 'color': '#4ea2f0' }, { 'id': 'N177', 'text': '上海冠润创业投资合伙企业（有限合伙）', 'color': '#4ea2f0' }, { 'id': 'N178', 'text': '深圳前海微众银行股份有限公司', 'color': '#4ea2f0' }, { 'id': 'N179', 'text': '北京英克必成科技有限公司', 'color': '#4ea2f0' }, { 'id': 'N180', 'text': '和泰人寿保险股份有限公司', 'color': '#4ea2f0' }, { 'id': 'N181', 'text': '北京知道创宇信息技术股份有限公司', 'color': '#4ea2f0' }, { 'id': 'N182', 'text': '常州哈酷那软件科技有限公司', 'color': '#4ea2f0' }, { 'id': 'N183', 'text': '腾讯云计算（北京）有限责任公司', 'color': '#4ea2f0' }], 'lines': [{ 'from': 'N2', 'to': 'N1', 'text': '' }, { 'from': 'N3', 'to': 'N1', 'text': '' }, { 'from': 'N4', 'to': 'N1', 'text': '' }, { 'from': 'N5', 'to': 'N1', 'text': '' }, { 'from': 'N1', 'to': 'N6', 'text': '出资:100%' }, { 'from': 'N1', 'to': 'N7', 'text': '出资:100%' }, { 'from': 'N1', 'to': 'N8', 'text': '出资:95%' }, { 'from': 'N1', 'to': 'N9', 'text': '出资:37.22%' }, { 'from': 'N1', 'to': 'N10', 'text': '出资:0%' }, { 'from': 'N1', 'to': 'N11', 'text': '出资:100%' }, { 'from': 'N11', 'to': 'N12', 'text': '出资:51%' }, { 'from': 'N11', 'to': 'N13', 'text': '出资:30%' }, { 'from': 'N11', 'to': 'N14', 'text': '出资:57.8%' }, { 'from': 'N1', 'to': 'N15', 'text': '出资:95%' }, { 'from': 'N15', 'to': 'N16', 'text': '出资:100%' }, { 'from': 'N15', 'to': 'N17', 'text': '出资:10%' }, { 'from': 'N15', 'to': 'N18', 'text': '出资:100%' }, { 'from': 'N15', 'to': 'N19', 'text': '出资:100%' }, { 'from': 'N1', 'to': 'N20', 'text': '出资:51%' }, { 'from': 'N20', 'to': 'N21', 'text': '出资:44%' }, { 'from': 'N1', 'to': 'N22', 'text': '出资:20.23%' }, { 'from': 'N1', 'to': 'N23', 'text': '出资:19.12%' }, { 'from': 'N1', 'to': 'N24', 'text': '出资:0%' }, { 'from': 'N1', 'to': 'N25', 'text': '出资:14.68%' }, { 'from': 'N1', 'to': 'N26', 'text': '出资:100%' }, { 'from': 'N1', 'to': 'N27', 'text': '出资:16.03%' }, { 'from': 'N1', 'to': 'N28', 'text': '出资:19.9%' }, { 'from': 'N1', 'to': 'N29', 'text': '出资:0%' }, { 'from': 'N1', 'to': 'N30', 'text': '出资:90%' }, { 'from': 'N1', 'to': 'N31', 'text': '出资:29%' }, { 'from': 'N31', 'to': 'N32', 'text': '出资:0.31%' }, { 'from': 'N1', 'to': 'N33', 'text': '出资:95%' }, { 'from': 'N33', 'to': 'N34', 'text': '出资:15%' }, { 'from': 'N33', 'to': 'N35', 'text': '出资:9.61%' }, { 'from': 'N1', 'to': 'N36', 'text': '出资:45%' }, { 'from': 'N1', 'to': 'N37', 'text': '出资:22.82%' }, { 'from': 'N1', 'to': 'N120', 'text': '出资:95%' }, { 'from': 'N120', 'to': 'N121', 'text': '出资:100%' }, { 'from': 'N120', 'to': 'N122', 'text': '出资:99%' }, { 'from': 'N120', 'to': 'N123', 'text': '出资:0%' }, { 'from': 'N120', 'to': 'N124', 'text': '出资:0%' }, { 'from': 'N120', 'to': 'N125', 'text': '出资:44.44%' }, { 'from': 'N1', 'to': 'N126', 'text': '出资:39.05%' }, { 'from': 'N1', 'to': 'N127', 'text': '出资:51%' }, { 'from': 'N1', 'to': 'N128', 'text': '出资:95%' }, { 'from': 'N1', 'to': 'N129', 'text': '出资:50%' }, { 'from': 'N1', 'to': 'N130', 'text': '出资:95%' }, { 'from': 'N130', 'to': 'N131', 'text': '出资:0%' }, { 'from': 'N1', 'to': 'N132', 'text': '出资:51%' }, { 'from': 'N132', 'to': 'N133', 'text': '出资:100%' }, { 'from': 'N132', 'to': 'N134', 'text': '出资:38%' }, { 'from': 'N132', 'to': 'N135', 'text': '出资:15%' }, { 'from': 'N132', 'to': 'N136', 'text': '出资:0%' }, { 'from': 'N132', 'to': 'N137', 'text': '出资:20%' }, { 'from': 'N132', 'to': 'N138', 'text': '出资:0%' }, { 'from': 'N132', 'to': 'N139', 'text': '出资:100%' }, { 'from': 'N1', 'to': 'N140', 'text': '出资:99%' }, { 'from': 'N1', 'to': 'N141', 'text': '出资:95%' }, { 'from': 'N141', 'to': 'N142', 'text': '出资:100%' }, { 'from': 'N1', 'to': 'N143', 'text': '出资:0%' }, { 'from': 'N1', 'to': 'N144', 'text': '出资:8%' }, { 'from': 'N1', 'to': 'N145', 'text': '出资:100%' }, { 'from': 'N145', 'to': 'N146', 'text': '出资:100%' }, { 'from': 'N145', 'to': 'N147', 'text': '出资:100%' }, { 'from': 'N1', 'to': 'N148', 'text': '出资:100%' }, { 'from': 'N1', 'to': 'N149', 'text': '出资:0%' }, { 'from': 'N1', 'to': 'N150', 'text': '出资:12.69%' }, { 'from': 'N150', 'to': 'N151', 'text': '出资:100%' }, { 'from': 'N1', 'to': 'N152', 'text': '出资:51%' }, { 'from': 'N152', 'to': 'N153', 'text': '出资:2.01%' }, { 'from': 'N1', 'to': 'N154', 'text': '出资:39%' }, { 'from': 'N1', 'to': 'N155', 'text': '出资:10.21%' }, { 'from': 'N1', 'to': 'N156', 'text': '出资:100%' }, { 'from': 'N156', 'to': 'N157', 'text': '出资:45%' }, { 'from': 'N1', 'to': 'N158', 'text': '出资:100%' }, { 'from': 'N1', 'to': 'N159', 'text': '出资:10.06%' }, { 'from': 'N1', 'to': 'N160', 'text': '出资:1.52%' }, { 'from': 'N1', 'to': 'N161', 'text': '出资:51%' }, { 'from': 'N161', 'to': 'N162', 'text': '出资:0%' }, { 'from': 'N161', 'to': 'N163', 'text': '出资:100%' }, { 'from': 'N1', 'to': 'N164', 'text': '出资:51%' }, { 'from': 'N164', 'to': 'N165', 'text': '出资:5%' }, { 'from': 'N1', 'to': 'N166', 'text': '出资:7.88%' }, { 'from': 'N1', 'to': 'N167', 'text': '出资:47.53%' }, { 'from': 'N1', 'to': 'N168', 'text': '出资:9%' }, { 'from': 'N1', 'to': 'N169', 'text': '出资:51.72%' }, { 'from': 'N1', 'to': 'N170', 'text': '出资:29%' }, { 'from': 'N170', 'to': 'N171', 'text': '出资:3.89%' }, { 'from': 'N170', 'to': 'N172', 'text': '出资:3.83%' }, { 'from': 'N170', 'to': 'N173', 'text': '出资:100%' }, { 'from': 'N170', 'to': 'N174', 'text': '出资:100%' }, { 'from': 'N170', 'to': 'N175', 'text': '出资:0%' }, { 'from': 'N170', 'to': 'N176', 'text': '出资:100%' }, { 'from': 'N170', 'to': 'N177', 'text': '出资:0%' }, { 'from': 'N170', 'to': 'N178', 'text': '出资:21.43%' }, { 'from': 'N1', 'to': 'N179', 'text': '出资:100%' }, { 'from': 'N179', 'to': 'N180', 'text': '出资:15%' }, { 'from': 'N179', 'to': 'N181', 'text': '出资:10.5%' }, { 'from': 'N179', 'to': 'N182', 'text': '出资:24.84%' }, { 'from': 'N179', 'to': 'N183', 'text': '出资:20%' }] };
            console.log(JSON.stringify(__graph_json_data));
            __graph_json_data.nodes.forEach(thisNode => {
              if (thisNode.text === '深圳市腾讯计算机系统有限公司') {
                thisNode.width = 300;
                thisNode.height = 100;
                thisNode.offset_x = -50; // 调整x偏移，让根节点看起来更居中
              }
              if (thisNode.text === '张志东' || thisNode.text === '陈一丹' || thisNode.text === '许晨晔' || thisNode.text === '马化腾') {
                thisNode.width = 100;
                thisNode.height = 80;
                // thisNode.offset_y = 80;
              }
              // 为节点《这个节点原本是没有子节点的》设置属性expandHolderPosition，使其在没有子节点的情况下也能显示【展开/收缩】按钮，当点击展开时动态加载子节点数据
              if (thisNode.text === '这个节点原本是没有子节点的') {
                thisNode.data = { asyncChild: true, loaded: false }; // 这是一个自定义属性，用来在后续判断如果点击了这个节点，则动态获取数据
                thisNode.expandHolderPosition = 'bottom';
                thisNode.expanded = false;
              }
            });
            setTimeout(() => {
              this.g_loading = false;
              this.$refs.seeksRelationGraph.setJsonData(__graph_json_data, (graphInstance) => {
                // 这些写上当图谱初始化完成后需要执行的代码
              });
            }, 1000);
          },
          onNodeExpand(node, e) {
            // 模拟动态加载数据
            if (node.data && node.data.asyncChild === true && node.data.loaded === false) {
              this.g_loading = true;
              node.data.loaded = true;
              setTimeout(() => {
                this.g_loading = false;
                const _new_json_data = {
                  nodes: [
                    { id: node.id + '-child-1', text: node.id + '-的子节点1' },
                    { id: node.id + '-child-2', text: node.id + '-的子节点2' },
                    { id: node.id + '-child-3', text: node.id + '-的子节点3' }
                  ],
                  lines: [
                    { from: node.id, to: node.id + '-child-1', text: '动态子节点' },
                    { from: node.id, to: node.id + '-child-2', text: '动态子节点' },
                    { from: node.id, to: node.id + '-child-3', text: '动态子节点' }
                  ]
                };
                this.$refs.seeksRelationGraph.appendJsonData(_new_json_data, (graphInstance) => {
                  // 这些写上当图谱初始化完成后需要执行的代码
                });
              }, 1000);
            }
          },
          onNodeClick(nodeObject, $event) {
            console.log(nodeObject)
            console.log($event)
          },
          onLineClick(lineObject, $event) {
            console.log(lineObject)
            console.log($event)
          },
          /**
           * 悬浮框显示
           * @param nodeObject
           * @param $event
           */
          showNodeTips(nodeObject, $event) {
            that.currentNode = nodeObject;
            const _base_position = that.$refs.test.getBoundingClientRect();
            console.log('showNodeMenus:', $event, _base_position);
            that.isShowNodeTipsPanel = true;
            that.nodeMenuPanelPosition.x = $event.clientX - _base_position.x + 10;
            that.nodeMenuPanelPosition.y = $event.clientY - _base_position.y + 10;
            // FxUI.objectUIAction.viewObject('AccountObj', '646ef69bcefe9a0001b8be20') // TODO bug无法解决 纷享Vue2版本(2.6.12)比RelationGraph需要的vue的版本(2.6.12以上，不包括2.6.12)低
          },

          /**
           * 悬浮框隐藏
           * @param nodeObject
           * @param $event
           */
          hideNodeTips(nodeObject, $event) {
            that.isShowNodeTipsPanel = false;
          }
        }
      })
      // this.myGraph = graph
      // graph.$mount() // 不要这个也挂载上了, 写这个的话组件会挂载两次， 会有两次mounted()然后this.showRelationGraph()会被调用两次
    },
  }
}
</script>

<style scoped>
/* 不知道为什么默认样式会让每个节点框中的文字右移，没有居中*/
/deep/ .rel-node-shape-1 .c-node-text{
  padding-left: 0;
  padding-right: 0;
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