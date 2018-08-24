<template>
  <div class="artboard">
    <div class="wrap" v-if="!noPage">
      <!-- 预览图显示 宽度多加右侧属性面板宽度 -->
      <div class="wrap-layer" :style="{
        width: previewDocument.width + 220 + 'px',
        height: previewDocument.height + 'px',
        background: 'url(' + previewUrl + ') no-repeat',
        position: 'relative'}">
        <!-- 创建选中图层的标注 -->
        <div id="sketch-marker" class="sketch-marker" :style="{
          visibility: markShow ? 'visible' : 'hidden',
          width: clickData.width + 'px',
          height: clickData.height + 'px',
          top: clickData.top + 'px',
          left: clickData.left + 'px'}">
          <b class="skm-w" :w="clickData.width + 'px'"></b><b class="skm-h" :h="clickData.height + 'px'"></b><span class="skm-tl"></span><span
            class="skm-tr"></span><span class="skm-bl"></span><span class="skm-br"></span></div>
        <!-- 创建虚线导航灯塔 -->
        <div id="skl-top" class="skl-top" :style="{
          display: markShow ? 'block' : 'none',
          top: clickData.top + 'px',
          width: previewDocument.width + 'px'}"></div>
        <div id="skl-bottom" class="skl-bottom" :style="{
          display: markShow ? 'block' : 'none',
          top: clickData.top + clickData.height + 'px',
          width: previewDocument.width + 'px'}"></div>
        <div id="skl-left" class="skl-left" :style="{
          display: markShow ? 'block' : 'none',
          left: clickData.left + 'px', 
          height: previewDocument.height + 'px'}"></div>
        <div id="skl-right" class="skl-right" :style="{
          display: markShow ? 'block' : 'none',
          left: clickData.left + clickData.width + 'px', 
          height: previewDocument.height + 'px'}"></div>
        <!-- 2个图层之间的标尺 -->
        <div id="skm-r-top" class="skm-rule skm-r-top" v-if="moveData.moveData" :style="{
          visibility: markShow && this.markData.top.visible ? 'visible' : 'hidden',
          left: markData.top.left + 'px',
          top: markData.top.top + 'px',
          height: markData.top.height + 'px'}"><span>{{markData.top.text}}px</span></div>
        <div id="skm-r-right" class="skm-rule skm-r-right" :style="{
          visibility: markShow && this.markData.right.visible ? 'visible' : 'hidden',
          top: markData.right.top + 'px',
          left: markData.right.left + 'px',
          width: markData.right.width + 'px'}"><span>{{markData.right.text}}px</span></div>
        <div id="skm-r-bottom" class="skm-rule skm-r-bottom" :style="{
          visibility: markShow && this.markData.bottom.visible ? 'visible' : 'hidden',
          left: markData.bottom.left + 'px',
          top: markData.bottom.top + 'px',
          height: markData.bottom.height + 'px'}"><span>{{markData.bottom.text}}px</span></div>
        <div id="skm-r-left" class="skm-rule skm-r-left" :style="{
          visibility: markShow && this.markData.top.visible ? 'visible' : 'hidden',
          top: markData.left.top + 'px',
          left: markData.left.left + 'px',
          width: markData.left.width + 'px'}"><span>{{markData.left.text}}px</span></div>
        <!-- 图层 -->
        <div class="art-layer" v-if="item.visible" v-for="(item, index) in tree" :key="item.index" :style="{
          width: item.width + 'px',
          height: item.height + 'px',
          left: item.left + 'px',
          top: item.top + 'px',
          position: 'absolute',
          zIndex: tree.length - index}" @click.stop="chooseNow(item)" @mouseover.stop="moveNow(item)">

        </div>
      </div>
    </div>
    <!-- 右侧属性面板 -->
    <transition name="custom-classes-transition" enter-active-class="animated bounceInRight" leave-active-class="animated bounceOutRight">
      <div class="panel panel-active" id="panel" v-if="markShow" @click.stop="panelClick">
        <dl class="pa-block"><dt><h2>{{clickData.name}}</h2></dt></dl>
        <dl class="pa-block"><dt><span>Position &amp; Size</span></dt><dd>
            <ul>
              <li><input :value="clickData.left + 'px'" readonly=""><label>X</label></li>
              <li><input :value="clickData.top + 'px'" readonly=""><label>Y</label></li>
              <li><input :value="clickData.width + 'px'" readonly=""><label>Width</label></li>
              <li><input :value="clickData.height + 'px'" readonly=""><label>Height</label></li>
            </ul>
          </dd>
        </dl>
        <dl class="pa-block" v-if="clickData.text && clickData.text.font"><dt><span>Content</span><em>Copy Success</em></dt><dd><textarea
              name="content" style="height: 19px;">{{clickData.name}}</textarea></dd>
        </dl>
        <dl class="pa-block" v-if="clickData.text && clickData.text.font"><dt><span>Font Size</span><em>Copy Success</em></dt><dd><textarea
              name="fontSize" style="height: 19px;">14px</textarea></dd>
        </dl>
        <dl class="pa-block"><dt><span>Code</span><em>Copy Success</em></dt><dd><textarea name="code" style="height: 110px;"
              :value="showCode"></textarea></dd>
        </dl>
        <dl class="pa-block pa-export" v-if="!(clickData.text && clickData.text.font)"><dt><span>Export</span></dt><dd>
            <ul>
              <li><label>Size</label><select name="size"><option select="" value="1x">1x</option></select></li>
              <li><label>Format</label><select name="format"><option value="png">png</option></select></li>
            </ul>
            <div class="export-btn"><a id="export-btn" target="_blank" :href="layerDir + clickData.name + '.png'" :download="clickData.name + '.png'" data-base="982C7C37-8BDE-4866-8316-09B03A2CA13A/43232933-2667-4A6E-B336-BFB3CC9F3619"
                class="panel-export">Export Activity Layer</a></div>
          </dd>
        </dl>
      </div>
    </transition>
    <Loading v-if="showLoad"></Loading>
    <div class="no-page" v-if="noPage">无页面信息，参数错误</div>
  </div>
</template>

<script>
  import Vue from 'vue'
  import Loading from './Loading.vue'
  import '../style/main.less'

  export default {
    name: 'Hello',
    props: {
      msg: String
    },
    data() {
      return {
        showLoad: true, // 是否显示loading
        noPage: false, //页面参数错误
        spaceNum: 0,
        tree: [], // psdjs解析出来的数据
        previewDocument: '', // 解析的图片宽高属性集合
        previewUrl: '', // 预览图片地址
        moveData: {}, // 当前鼠标移动到的图层
        clickData: {}, // 当前点击选中图层
        markData: { // 标记尺寸
          top: {},
          left: {},
          bottom: {},
          right: {}
        },
        markShow: false // 是否有标注 默认false
      }
    },
    components: {
      Loading
    },
    computed: {
      /**
       * 显示code
       */
      showCode() {
        let code = ''
        if (this.clickData.text && this.clickData.text.font) {
          let opacity = this.clickData.text.font.colors[0].pop() / 255
          code += ('font-family: ' + this.clickData.text.font.name + '\r\n'
            + 'font-size: ' + this.clickData.text.font.sizes[0] + 'px\r\n'
            + 'color: rgba(' + this.clickData.text.font.colors[0] + ',' + opacity + ')\r\n')
            + 'text-align: ' + this.clickData.text.font.alignment[0] + '\r\n'
        } else {
          code += ('width: ' + this.clickData.width + 'px\r\n'
            + 'height: ' + this.clickData.height + 'px\r\n'
          )
        }
        if (this.clickData.opacity < 1) {
          code += ('opacity: ' + this.clickData.opacity + '\r\n')
        }

        return code
      }
    },
    mounted() {
      let url = this.$route.query.url
      let preview = this.$route.query.preview
      console.warn('获取数据', url, preview)
      if (url && preview) {
        document.body.addEventListener('click', this.bodyListener, false)
        Vue.axios.get('http://localhost:3000/getPsdJson', {
          params: {
            id: url
          }
        }).then((response) => {
          this.showLoad = false
          this.previewUrl = response.data.preview // 预览图
          this.layerDir = response.data.layerDir // 切图url前缀
          this.previewDocument = response.data.tree.document
          // 数据扁平化
          this.traversalPsdTree(response.data.tree.children, 1)
          console.warn(Object.keys(this.tree), this.tree)
          this.tree.forEach((item) => {
            //console.warn('object',item.type);
            /* if (!(item.text && item.text.font)) {
              item.width = item.width - 3
              item.height = item.height - 3
              item.left = item.left + 1
              item.top = item.top + 1
            } */

          })

        }).catch(() => {
          // 参数错误页面
          this.noPage = true
          this.showLoad = false
        })
      } else {
        // 参数错误页面
        this.noPage = true
        this.showLoad = false
      }

    },
    beforeDestroy() {
      document.body.removeEventListener('click', this.bodyListener)
    },
    methods: {
      bodyListener(evt) {
        console.error('object', evt)
        this.markShow = false
      },
      /**
       * 鼠标移动到的图层 
       */
      moveNow(data) {
        // 是否有点击选中
        if (Object.keys(this.clickData).length === 0) {
          return
        }
        const isDiff = this.diff(data, this.clickData)
        //console.log('isDiff', isDiff)
        if (isDiff) {
          this.moveData = {}
        } else {
          this.moveData = {
            moveData: data,
            clickData: this.clickData
          }
          this.showRuler()
        }
        //console.log(this.moveData)
      },
      /**
       * 当前选中图层
       */
      chooseNow(data) {
        console.info('chooseNow', data)
        this.markShow = true
        this.changeRulerState('none', {})
        this.clickData = data
      },
      /**
       * 显示标记
       */
      showRuler() {
        let buffer = 0;
        let data = {
          //选中节点宽
          sw: this.moveData.clickData.width,
          //选中节点高
          sh: this.moveData.clickData.height,
          //选中节点x坐标
          sx: this.moveData.clickData.left,
          //选中节点y坐标
          sy: this.moveData.clickData.top,
          //mouseover节点宽
          tw: this.moveData.moveData.width,
          //mouseover节点高
          th: this.moveData.moveData.height,
          //mouseover节点x坐标
          tx: this.moveData.moveData.left,
          //mouseover节y坐标
          ty: this.moveData.moveData.top
        }
        //鼠标经过的图层在选中图层上方(T1 T2的位置)
        if (data.sy - (data.ty + data.th) >= buffer) {
          if (data.sx - (data.tx + data.tw) >= buffer) {
            //[上]左 显示向下，向右的标尺
            this.changeRulerState('bottom-right', data);
          } else if (data.tx - (data.sx + data.sw) >= buffer) {
            //[上]右 显示向下，向左的标尺
            this.changeRulerState('bottom-left', data);
          } else {
            this.changeRulerState('bottom', data);
          }
        }

        //鼠标经过的图层在选中图层下方(T1 T2的位置)
        if (data.ty - (data.sy + data.sh) > buffer) {
          if (data.sx - (data.tx + data.tw) >= buffer) {
            //[下]左 显示向上，向右的标尺
            this.changeRulerState('top-right', data);
          } else if (data.tx - (data.sx + data.sw) >= buffer) {
            //[下]右 显示向上，向左的标尺
            this.changeRulerState('top-left', data);
          } else {
            this.changeRulerState('top', data);
          }
        }

        //鼠标经过的图层与选中图层映射到垂直线上有交集只显示水平的标尺
        if ((data.ty >= data.sy && data.ty <= (data.sy + data.sh)) || (data.sy >= data.ty && data.sy <= (data.ty + data.th))) {
          if (data.sx - (data.tx + data.tw) >= buffer) {
            this.changeRulerState('right', data);
          } else if (data.tx - (data.sx + data.sw) >= buffer) {
            this.changeRulerState('left', data);
          } else {
            //鼠标图层(t)包含选中图层(s)(标尺总是由target指向selected)
            this.changeRulerState('container-in', data);
          }
        }

        //鼠标经过的图层与选中图层映射到水平线上有交集只显示垂直的标尺
        if (data.tx >= data.sx && data.tx <= (data.sx + data.sw)) {
          if (data.sy - (data.ty + data.th) >= buffer) {
            this.changeRulerState('bottom', data);
          } else if (data.ty - (data.sy + data.sh) >= buffer) {
            this.changeRulerState('top', data);
          } else {
            //被选中图层(s)包含鼠标图层(t)(标尺总是由target指向selected)
            this.changeRulerState('container-out', data);
          }
        }
      },
      /**
       * 显示标尺，标尺总是由target指向selected
       */
      changeRulerState(type, data) {
        this.markData.top.visible = false
        this.markData.left.visible = false
        this.markData.right.visible = false
        this.markData.bottom.visible = false
        if (type == 'none') { return false }
        //向上的标尺
        if (type.indexOf('top') > -1) {
          this.markData.top = {
            left: (data.tx + data.tw / 2),
            top: (data.sy + data.sh + 2),
            height: parseInt((data.ty - data.sy - data.sh - 4), 10),
            text: parseInt((data.ty - data.sy - data.sh), 10),
            visible: true
          }
        }

        //向左的标尺
        if (type.indexOf('left') > -1) {
          this.markData.left = {
            top: (data.ty + data.th / 2),
            left: (data.sx + data.sw + 2),
            width: parseInt((data.tx - data.sx - data.sw - 4), 10),
            text: parseInt((data.tx - data.sx - data.sw), 10),
            visible: true
          }
        }

        //向右的标尺
        if (type.indexOf('right') > -1) {
          this.markData.right = {
            top: (data.ty + data.th / 2),
            left: (data.tx + data.tw + 2),
            width: parseInt((data.sx - data.tx - data.tw - 4), 10),
            text: parseInt((data.sx - data.tx - data.tw), 10),
            visible: true
          }
        }

        //向下的标尺
        if (type.indexOf('bottom') > -1) {
          this.markData.bottom = {
            left: (data.tx + data.tw / 2),
            top: (data.ty + data.th + 2),
            height: parseInt((data.sy - data.ty - data.th - 4), 10),
            text: parseInt((data.sy - data.ty - data.th), 10),
            visible: true
          }
        }

        //鼠标图层(t)包含选中图层(s)
        if (type == 'container-in') {
          this.markData.top = {
            left: (data.sx + data.sw / 2),
            top: (data.ty + 1),
            height: parseInt((data.sy - data.ty - 3), 10),
            text: parseInt((data.sy - data.ty), 10),
            visible: true
          }
          this.markData.bottom = {
            left: (data.sx + data.sw / 2),
            top: (data.sy + data.sh + 2),
            height: this.validateNum(data.ty + data.th - data.sy - data.sh - 2),
            text: this.validateNum(data.ty + data.th - data.sy - data.sh),
            visible: true
          }
          this.markData.left = {
            top: (data.sy + data.sh / 2),
            left: (data.tx + 1),
            width: parseInt(this.validateNum(data.sx - data.tx - 3), 10),
            text: parseInt((data.sx - data.tx), 10),
            visible: true
          }
          this.markData.right = {
            top: (data.sy + data.sh / 2),
            left: (data.sx + data.sw + 2),
            width: this.validateNum(data.tx + data.tw - data.sx - data.sw - 3),
            text: this.validateNum(data.tx + data.tw - data.sx - data.sw),
            visible: true
          }
        }
        //被选中图层(s)包含鼠标图层(t)
        if (type == 'container-out') {
          this.markData.top = {
            left: (data.tx + data.tw / 2),
            top: (data.sy + 1),
            height: this.validateNum(data.ty - data.sy - 3),
            text: this.validateNum(data.ty - data.sy),
            visible: true
          }
          this.markData.bottom = {
            left: (data.tx + data.tw / 2),
            top: (data.ty + data.th + 2),
            height: this.validateNum(data.sy + data.sh - data.ty - data.th - 3),
            text: this.validateNum(data.sy + data.sh - data.ty - data.th),
            visible: true
          }
          this.markData.left = {
            top: (data.ty + data.th / 2),
            left: (data.sx + 1),
            width: this.validateNum(data.tx - data.sx - 3),
            text: this.validateNum(data.tx - data.sx),
            visible: true
          }
          this.markData.right = {
            top: (data.ty + data.th / 2),
            left: (data.tx + data.tw + 2),
            width: this.validateNum(data.sx + data.sw - data.tx - data.tw - 3),
            text: this.validateNum(data.sx + data.sw - data.tx - data.tw),
            visible: true
          }
        }
      },
      validateNum: function (num) {
        return num < 0 ? parseInt(Math.abs(num), 10) : parseInt(num, 10);
      },
      /**
       * 右侧属性面板点击
       */
      panelClick() {

      },
      /**
       * psdTree扁平化方法
       *
       */
      createSpaceStr(argument) {
        let spaceStr = ''
        for (let i = 0; i < this.spaceNum; i++) {
          spaceStr += '->'
        }
        return spaceStr
      },
      /**
       * psdTree扁平化入口
       *
       */
      traversalPsdTree(tree, floor) {
        let spaceStr = this.createSpaceStr()
        for (let i = 0; i < tree.length; i++) {
          let node = tree[i]
          if (node.visible) {
            if (node.type == 'group') {
              this.spaceNum++
              let next = floor + 1
              this.traversalPsdTree(node.children, next)
            } else {
              // console.log(spaceStr + 'floor : ' + floor + '   node  : ' + i + ' ' + node.type)
              this.tree.push(node)
            }
          }
        }
        this.spaceNum--
      },
      /**
       *  对比
       */
      diff(obj1, obj2) {
        let o1 = obj1 instanceof Object
        let o2 = obj2 instanceof Object
        if (!o1 || !o2) { /*  判断不是对象  */
          return obj1 === obj2
        }

        if (Object.keys(obj1).length !== Object.keys(obj2).length) {
          return false
          // Object.keys() 返回一个由对象的自身可枚举属性(key值)组成的数组,例如：数组返回下表：let arr = ["a", "b", "c"];console.log(Object.keys(arr))->0,1,2;
        }

        for (let attr in obj1) {
          let t1 = obj1[attr] instanceof Object
          let t2 = obj2[attr] instanceof Object
          if (t1 && t2) {
            return this.diff(obj1[attr], obj2[attr])
          } else if (obj1[attr] !== obj2[attr]) {
            return false
          }
        }
        return true
      }
    }
  }
</script>
<style>
  .no-page {
    margin-top: 100px;
  }
</style>