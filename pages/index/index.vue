<template>
	<view class="content">
		<!-- #ifdef APP-PLUS || H5 -->
    <view class="">
      {{ option.title.text }}
    </view>
    <!-- <scroll-view scroll-x="true" style="height: 500px"> -->
      <view @click="lib.handleClick" :prop="option" :change:prop="lib.updateDate" id="libInstance" class="lib"></view>
    <!-- </scroll-view> -->
		<button @click="changeOption">更新数据</button>
		<!-- #endif -->
		<!-- #ifndef APP-PLUS || H5 -->
		<view>非 APP、H5 环境不支持</view>
		<!-- #endif -->
	</view>
</template>

<script>
	export default {
		data() {
			return {
				option: {
					title: {
						text: 'D3 jsPlumb 入门示例'
					},
          blocks: [
            {
              title: '结构1',
              x: 0,
              y: 0,
              id: 'b1',
              anchors: ['Bottom']
            },
            {
              title: '结构2',
              x: 100,
              y: 100,
              id: 'b2',
              anchors: ['Bottom']
            },
            {
              title: '结构3',
              x: 200,
              y: 200,
              id: 'b3',
              anchors: ['Bottom']
            }
          ]
				}
			}
		},
		onLoad() {
		},
		methods: {
			changeOption() {
				const data = this.option.series[0].data
				// 随机更新示例数据
				data.forEach((item, index) => {
					data.splice(index, 1, Math.random() * 40)
				})
			},
			onViewClick(options) {
				console.log(options)
			}
		}
	}
</script>

<script module="lib" lang="renderjs">
	let myChart
	export default {
		mounted() {
      let script = document.createElement('script')
      script.src = 'static/d3.v5.js'
      const d3Promise = new Promise((resolve, reject) => {
        script.onload = () => resolve()
      })
      document.head.appendChild(script)
      script = document.createElement('script')
      script.src = 'static/jsPlumb.v2.6.8.js'
      const jsPlumbPromise = new Promise((resolve, reject) => {
        script.onload = () => {
          window.jsPlumb.ready(() => resolve())
        }
      })
      document.head.appendChild(script)
      Promise.all([d3Promise, jsPlumbPromise]).then(res => this.initLib())
		},
		methods: {
			initLib() {
        const d3 = window.d3
        const jsPlumb = window.jsPlumb
        d3.select('#libInstance')
          .selectAll('div')
            .data(this.option.blocks)
            .enter()
            .append('div')
            .classed('block', true)
            .attr('id', function (d) {
              return d.id
            })
            .style('left', function (d) {
              return d.x + 'px'
            })
            .style('top', function (d) {
              return d.y + 'px'
            })
            .text(function (d) {
              return '标题' + d.title
            })
        var endpointOpts = {
          isSource: true,
          isTarget: true,
          connector: ['Bezier'],
          endpoint: 'Dot',
          paintStyle: {
            fill: 'white',
            outlineStroke: 'blue',
            strokeWidth: 3
          },
          hoverPaintStyle: {
            outlineStroke: 'lightblue'
          },
          connectorStyle: {
            outlineStroke: 'green',
            strokeWidth: 1
          },
          connectorHoverStyle: {
            strokeWidth: 2
          }
        }
        this.option.blocks.forEach((item) => {
          jsPlumb.draggable(item.id, { containment: 'parent', grid: [10, 10] })
          jsPlumb.addEndpoint(item.id, { anchors: item.anchors }, endpointOpts)
        })
        jsPlumb.setContainer('libInstance')
        // jsPlumb.connect({
        //   source: 'b1',
        //   target: 'b2',
        //   endpoint: 'Dot',
        //   paintStyle: { stroke: 'lightgray', strokeWidth: 3 },
        //   endpointStyle: { fill: 'lightgray', outlineStroke: 'darkgray', outlineWidth: 2 }
        // })
        jsPlumb.bind('click', function (conn, originalEvent) {
          uni.showModal({
            content: '确定删除所点击的链接吗？',
            success: () => {
              jsPlumb.deleteConnection(conn)
              jsPlumb.remove('b1')
            }
          })
        })
        jsPlumb.bind('contextmenu', function () {
          arguments
        })
			},
			updateDate(newValue, oldValue, ownerInstance, instance) {
				// 监听 service 层数据变更
				// myChart.setOption(newValue)
        console.log(newValue, oldValue, ownerInstance, instance)
			},
			handleClick(event, ownerInstance) {
				// 调用 service 层的方法
				ownerInstance.callMethod('onViewClick', {
					test: 'test'
				})
			}
		}
	}
</script>

<style lang="scss" scoped>
	.content {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
	}

	.lib {
		margin-top: 100px;
    background: red;
		width: 100%;
		height: 400px;
    width: 100%;
    position: relative;
    ::v-deep .block {
      position: absolute;
      text-align: center;
      line-height: 60rpx;
      height: 60rpx;
      width: 240rpx;
      border: 1px solid;
    }
	}
</style>
