<template>
    <div id="joint-app" class="joint-app joint-theme-modern">
        <div class="joint-app-header">
        </div>
        <div class="joint-app-body">
            <div id="joint-stencil" class="stencil-container"></div>
            <div id="joint-paper" class="paper-container" @mousemove='updateXY'></div>
            <div class="inspector-container">
                <div class="form-horizontal" style="padding: 10px">
                    <h1>编辑图形</h1>
                    <h2>Text</h2>
                    <div class="input-group mb-3 input-group-lg ">
                        <div style="font-size: 15px;margin-top: 10px;margin-right: 10px">文字大小</div>
                        <input type="text" class="form-control" placeholder="文字大小" v-model="shapAttrs.text.fontSize"
                               @change="changeTextFontSize(shapAttrs.cell,shapAttrs.text.fontSize)">
                    </div>
                    <div class="input-group mb-3 input-group-lg ">
                        <div style="font-size: 15px;margin-top: 10px;margin-right: 10px">文字内容</div>
                        <input type="text" class="form-control" placeholder="文字内容" v-model="shapAttrs.text.content"
                               @change="changeTextContent(shapAttrs.cell,shapAttrs.text.content)">
                    </div>
                    <div class="form-group input-group input-group-lg">
                        <div style="font-size: 15px;margin-top: 10px;margin-right: 10px">文字颜色</div>
                        <input type="text" class="form-control" placeholder="文字颜色" v-model="shapAttrs.text.color"
                               @change="changeTextColor(shapAttrs.cell,shapAttrs.text.color)">
                    </div>
                    <h2>Shap</h2>
                    <div class="input-group mb-3 input-group-lg ">
                        <div style="font-size: 15px;margin-top: 10px;margin-right: 10px">高度</div>
                        <input type="text" class="form-control" placeholder="高度" v-model="shapAttrs.height"
                               @change="changeHeight(shapAttrs.cell,shapAttrs.height)">
                    </div>
                    <div class="form-group input-group input-group-lg">
                        <div style="font-size: 15px;margin-top: 10px;margin-right: 10px">宽度</div>
                        <input type="text" class="form-control" placeholder="宽度" v-model="shapAttrs.width"
                               @change="changeWidth(shapAttrs.cell,shapAttrs.width)">
                    </div>
                    <div class="form-group input-group input-group-lg">
                        <div style="font-size: 15px;margin-top: 10px;margin-right: 10px">颜色</div>
                        <input type="text" class="form-control" placeholder="颜色" v-model="shapAttrs.color"
                               @change="changeColor(shapAttrs.cell,shapAttrs.color)">
                    </div>
                </div>
            </div>
        </div>
    </div>

</template>

<script>
import '../../static/css/joint.css'
require('lodash')
const joint = require('jointjs')
require('../../static/js/backbone.js')

export default {
  props: {},
  data () {
    return {
      x: 0,
      y: 0,
      active: true,
      graph: null,
      shapAttrs: {
        cell: null,
        height: 60,
        width: 60,
        color: '',
        text: {
          content: '',
          color: '',
          fontSize: '10'
        }
      },
      linkAttrs: {
        cell: null,
        text: '',
        color: ''
      }

    }
  },
  mounted: function () {
    // this.shaps()
    this.init()
    this.initComponent()
  },
  computed: {},
  created () {
    // this.init()
  },
  methods: {
    updateXY: function (event) {
      this.x = event.offsetX
      this.y = event.offsetY
    },
    changeTextFontSize (cell, fontSize) {
      cell.attr('label/fontSize', fontSize)
    },
    changeTextContent (cell, content) {
      cell.attr('label/text', content)
    },
    changeTextColor (cell, color) {
      cell.attr('label/fill', color)
    },
    changeHeight (cell, height) {
      cell.size(this.shapAttrs.width, height)
    },
    changeWidth (cell, width) {
      cell.size(width, this.shapAttrs.height)
    },
    changeColor (cell, color) {
      cell.attr('body/fill', color)
    },
    init () {
      let _this = this
      this.graph = new joint.dia.Graph()
      this.gateTemplate = new joint.shapes.devs.Model({
        position: {
          x: 0,
          y: 0
        },
        size: {
          width: 60,
          height: 50
        },
        // portMarkup: '<rect class="joint-port-body" width="10" height="3" style="fill:black" />',
        // portLabelMarkup: '<text class="port-label joint-port-label" font-size="10" y="0" fill="#000" /> ',
        ports: {
          groups: {
            'in': {
              position: 'top',
              attrs: {
                '.port-body': {
                  magnet: 'passive'
                },
                '.joint-port-body': {
                  x: -10
                }
              },
              label: {
                position: {
                  args: {
                    x: 18
                  }
                }
              }
            },
            'out': {
              position: 'bottom',
              z: -1,
              attrs: { // 这里如果不写入stroke以及stroke-opacity的话，不会出现高亮的效果
                'circle': {
                  magnet: true, // 控制改锚点是否可以被连接
                  stroke: '#5cc05c', // 锚点的边的填充
                  fill: '#5cc05c',
                  // r: (type=='TASK'||type=='FILTER') ? 10 : 12,
                  // "ref-y": type=='TASK'?14:0,
                  r: 12,
                  'ref-y': 0,
                  'stroke-opacity': 0.5 // 高亮的时候，边的颜色
                }
              },
              label: {
                position: {
                  args: {
                    x: -23
                  }
                }
              }
            }
          }
        },
        attrs: {
          '.label': {
            'type': 'primary',
            fontSize: 12,
            'ref-x': 0.5,
            'ref-y': 0.05
          }
        }
      })
      var paper = new joint.dia.Paper({
        el: $('#joint-paper'),
        width: 790,
        height: 590,
        gridSize: 10, // 图形元素移动步进像素
        drawGrid: true, // 显示步进点
        /* background: {
            color: 'rgba(0, 0, 0, 0.1)'
        }, */
        model: this.graph,
        // defaultLink: new joint.dia.Link({
        //     attrs: {'.marker-target': {d: 'M 10 0 L 0 5 L 10 10 z'}}
        // }),
        defaultLink: new joint.shapes.logic.Wire({
          connector: {name: 'jumpover'} // 当连线交叉时，跳过
        }),
        linkPinning: false, // 不允许连线到空白处
        // 距离节点连接点25像素时自动连接上
        snapLinks: {
          radius: 1
        },
        elementView: joint.dia.ElementView.extend({
          // 双击删除
          pointerdblclick: function (evt, x, y) {
            joint.dia.CellView.prototype.pointerdblclick.apply(this, arguments)
            this.notify('element:pointerdblclick', evt, x, y)
            this.model.remove()
          }
        }),
        linkView: joint.dia.LinkView.extend({
          // 双击删除
          pointerdblclick: function (evt, x, y) {
            joint.dia.CellView.prototype.pointerdblclick.apply(this, arguments)
            this.notify('link:pointerdblclick', evt, x, y)
            this.model.remove()
          }
        })
        // 验证连线是否允许
        // validateConnection: function (viewSource, magnetSource, viewTarget, magnetTarget, end, viewLink) {
        //   if (end === 'target') {
        //     // 连线目标必须时一个"in"类型连接点
        //     if (!magnetTarget || !magnetTarget.getAttribute('port-group') || magnetTarget.getAttribute('port-group').indexOf('in') < 0) {
        //       return false
        //     }
        //
        //     // 检查连接点是否已经有线接入
        //     var portUsed = this.model.getLinks().some(function (link) {
        //       return (link.id !== viewLink.model.id &&
        //         link.get('target').id === viewTarget.model.id &&
        //         link.get('target').port === magnetTarget.getAttribute('port'))
        //     })
        //
        //     return !portUsed
        //   } else { // end === 'source'
        //     // 连线起始点必须时一个"out"类型连接点
        //     return magnetSource && magnetSource.getAttribute('port-group') && magnetSource.getAttribute('port-group').indexOf('out') >= 0
        //   }
        // }
      })

      // var stencil = new joint.ui.Stencil({
      //   paper: paper,
      //   width: 200,
      //   height: 300
      // })
      //
      // $('#joint-stencil').append(
      //   stencil.render().el
      // )

      var delButton = new joint.shapes.standard.TextBlock()
      delButton.resize(10, 10)
      delButton.position(0, 0)
      delButton.attr('label/text', 'X')

      paper.on({
        // 'blank:pointerdown': function(elementView, evt) {
        //     if (this.currentEle) {
        //       console.log(this.currentEle);
        //       this.currentEle.unhighlight();
        //       this.currentEle = null;
        //     }
        // },
        'element:pointerdown': function (elementView, evt) {
          if (elementView.model.attr('.label/type') === 'primary') {
            var type = elementView.model.attr('.label/text')
            if (type === 'And') {
              _this.graph.addCell(_this.genAndPr().translate(20, 20))
            } else if (type === 'Or') {
              _this.graph.addCell(_this.genOrPr().translate(20, 120))
            } else if (type === 'Not') {
              _this.graph.addCell(_this.genNotPr().translate(20, 220))
            }
            // 挪到图层的上层
            elementView.model.toFront()
          } else if (elementView.model.attr('.label/type') === 'instance') {
            evt.data = elementView.model.position()
          }
        },
        'element:pointerup': function (elementView, evt, x, y) {
          if (elementView.model.attr('.label/type') === 'primary') {
            if (elementView.model.position().x > 105) {
              var type = elementView.model.attr('.label/text')
              if (type === 'And') {
                _this.graph.addCell(_this.genAnd().translate(elementView.model.position().x, elementView.model.position().y))
              } else if (type === 'Or') {
                _this.graph.addCell(_this.genOr().translate(elementView.model.position().x, elementView.model.position().y))
              } else if (type === 'Not') {
                _this.graph.addCell(_this.genNot().translate(elementView.model.position().x, elementView.model.position().y))
              }
            }
            _this.graph.removeCells(elementView.model)
          } else {
            if (elementView.model.position().x < 110) {
              elementView.model.position(evt.data.x, evt.data.y)
            }
          }
        },
        'element:mouseover': function (elementView, evt) {
          if (elementView.model.attr('.label/type') === 'instance') {
            console.log(elementView)
            elementView.highlight()
            elementView.model.embed(delButton.clone().translate(elementView.model.position().x, elementView.model.position().y))
          }
        },
        'element:mouseout': function (elementView, evt) {
          if (elementView.model.attr('.label/type') === 'instance') {
            elementView.unhighlight()
          }
        },

        'element:pointerdblclick': function (elementView, evt) {
          if (elementView.model.attr('.label/type') === 'instance') {
            evt.data = elementView.model.position()
            elementView.model.remove()
          }
        },
        // 实现画布拖拽
        'blank:pointerdown': function (evt, x, y) {
          this.startX = x
          this.startY = y
          this.origin = paper.options.origin
          this.startMove = true
        },
        'blank:pointermove': function (evt, x, y) {
          if (this.startMove) {
            var diffX = x - this.startX + this.origin.x
            var diffY = y - this.startY + this.origin.y
            paper.translate(diffX, diffY)
          }
        },
        'blank:pointerup': function (evt, x, y) {
          this.startMove = false
        }
      })

      var separator = new joint.shapes.standard.Polyline()
      separator.resize(5, 400)
      separator.position(95, 0)
      // separator.attr('body/refPoints', '115,0 115,400 120,400 120,0');
      separator.addTo(this.graph)
    },
    initComponent () {
      this.graph.addCell(this.genAndPr().translate(20, 20))
      this.graph.addCell(this.genOrPr().translate(20, 120))
      this.graph.addCell(this.genNotPr().translate(20, 220))
    },
    genAndPr () {
      return this.gateTemplate.clone().set('inPorts', ['in1']).set('outPorts', ['out']).attr('.label/text', 'And')
    },
    genAnd () {
      return this.genAndPr().attr('.label/type', 'instance')
    },
    genOrPr () {
      return this.gateTemplate.clone().set('inPorts', ['in1']).set('outPorts', ['out']).attr('.label/text', 'Or')
    },
    genOr () {
      return this.genOrPr().attr('.label/type', 'instance')
    },
    genNotPr () {
      return this.gateTemplate.clone().set('inPorts', ['in']).set('outPorts', ['out']).attr('.label/text', 'Not')
    },
    genNot () {
      return this.genNotPr().attr('.label/type', 'instance')
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
    .joint-app-header {
        position: relative;
        width: 100%;
    }

    /*#joint-stencil {
        background: #146DFF;
    }*/
    html, body, .joint-app {
        position: relative;
        width: 100%;
        height: 100%;
        box-sizing: border-box;
        margin: 0;
        padding: 0;
    }

    .joint-app-body {
        position: relative;
        height: -moz-calc(100% - 60px);
        height: -webkit-calc(100% - 60px);
        height: calc(100% - 60px);
    }

    .paper-container {
        position: absolute;
        top: 0;
        height: 100%;
        overflow: hidden;
        box-sizing: border-box;
        left: 240px;
        right: 240px;
    }

    .inspector-container {
        position: absolute;
        top: 0;
        right: 0;
        bottom: 120px;
        width: 240px;
        box-sizing: border-box;
        height: 590px;
        background-color: #9093B1;
    }

    .joint-inspector .group {
        overflow: hidden;
        padding: 0;
        padding-bottom: 10px;
    }

    .joint-inspector .group > .group-label {
        position: relative;
        padding: 5px 4px;
        margin-top: 0;
        margin-bottom: 0;
        cursor: pointer;
        -webkit-user-select: none;
        -moz-user-select: none;
        -ms-user-select: none;
        user-select: none;
    }

</style>
