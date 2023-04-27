<template>
  <div class="home">
    <div class="origin-img-wrapper" style="flex: 1;">
      <img src="../assets/canvas-img.jpg" ref="originImgRef">
    </div>

    <div class="right">
      <div style="margin-right: 30px;position: relative;">
        <div class="tools-wrapper" style="margin-bottom: 50px;">
          <div
            v-for="item in tools"
            :key="item"
            class="tool-item"
            :class="{'tool-item-selected': curtool === item}"
            @click="setUsingTool(item)"
          >
            {{ item }}
          </div>
        </div>
        
        <div style="margin-bottom: 50px;">
          <div style="margin-bottom: 20px;">当前步数：{{ canvasDataRecords.length }}</div>
          <div class="save-btn" :class="{'disabled-btn': canvasDataRecords.length === 1}" @click="drawPreData">
            回退
          </div>
          <div class="save-btn" @click="resetCanvasImg">
            重置
          </div>
          <div class="save-btn" @click="saveCanvasImg">
            保存
          </div>
        </div>

        <div v-show="showWidthSelector" style="position: absolute;left: 50%;transform: translateX(-50%);">
          <div class="change-width-wrapper" style="margin-bottom: 50px;">
            <div
              class="width-item"
              :class="{'width-item-selected': item.name === selectedWidth.name}"
              v-for="item in widthLevel"
              :key="item.name"
              @click="selectedWidth = item"
            >
              {{ item.name }}
            </div>
          </div>
          <div class="color-selector-wrapper">
            <div
              class="color-item"
              :class="{'color-item-selected': item === drawcolor}"
              v-for="item in colorArr"
              :style="{backgroundColor: item}"
              :key="item"
              @click="drawcolor = item"
            >
              {{ item.name }}
            </div>
          </div>
        </div>
      </div>

      <div class="canvas-area">
        <canvas class="my-canvas" id="myCanvas"></canvas>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'HomeView',
  data() {
    return {
      tools: [
        'pencil', // 画笔
        'text', // 文字
      ],
      curtool: 'pencil',
      mouseDown: false,
      widthLevel: [
        { name: '极细',value: 1 },
        { name: '细',value: 2 },
        { name: '中',value: 3 },
        { name: '粗',value: 4 },
      ],
      selectedWidth: {},
      showWidthSelector: false,
      colorArr: ['#ff0000', '#FFFF00', '#00FF31'],
      drawcolor: '#ff0000',
      canvasDataRecords: [],
      currentRecordIndex: 0,
      lastCanvasData: null,
      canvasElement: null,
      startPointX: 0,
      startPointY: 0,
    }
  },

  watch: {
    curtool: {
      handler(newVal) {
        if(newVal === 'pencil') {
          this.showWidthSelector = true
        }
        else {
          this.showWidthSelector = false
        }
      },
      immediate: true
    },
  },

  created() {
    this.selectedWidth = this.widthLevel[0]
  },

  mounted() {
    this.initCanvasBgImg()
  },

  methods: {
    initCanvasBgImg() {
      const canvas = document.getElementById('myCanvas')
      this.canvasElement = canvas
      const ctx = canvas.getContext('2d')
      this.canvasContext = ctx
      const img = this.$refs.originImgRef
      img.onload = () => {
        canvas.width = img.width
        canvas.height = img.height
        ctx.drawImage(img, 0, 0, img.width , img.height)
        this.saveLastData()
      }
      this.addCanvasEvent()
    },

    resetCanvasImg() {
      this.canvasDataRecords = [this.canvasDataRecords[0]]
      this.drawIndexData(0)
    },

    setUsingTool(tool) {
      this.curtool = tool
    },

    changeColor(color) {
      this.drawcolor = color
    },

    // 将最后一次的画布数据绘制到画布上
    drawLastData() {
      this.canvasContext.putImageData(this.lastCanvasData, 0, 0)
    },

    // 将指定历史记录下标的画布数据绘制到画布上
    drawIndexData(index) {
      const data = this.canvasDataRecords[index]
      this.canvasContext.putImageData(data, 0, 0)
    },

    // 撤回上一步的画布数据
    drawPreData() {
      if(this.canvasDataRecords.length === 1) {
        return
      }
      this.canvasDataRecords.pop()
      this.lastCanvasData = this.canvasDataRecords[this.canvasDataRecords.length-1]
      this.drawLastData()
    },

    // 保存最后一次画布数据，并添加到数据历史记录中
    saveLastData() {
      this.lastCanvasData = this.canvasContext.getImageData(0, 0, this.canvasElement.width, this.canvasElement.height)
      this.canvasDataRecords.push(this.lastCanvasData)
      this.drawLastData()
    },

    // 添加canvas事件
    addCanvasEvent() {
      const canvas = this.canvasElement
      canvas.addEventListener('mousedown', this.handleMouseDown)
      canvas.addEventListener('mousemove', this.handleMouseMove)
      canvas.addEventListener('mouseup', this.handleMouseUp)
    },

    removeCanvasEvent() {
      const canvas = this.canvasElement
      canvas.removeEventListener('mousedown', this.handleMouseDown)
      canvas.removeEventListener('mousemove', this.handleMouseMove)
      canvas.removeEventListener('mouseup', this.handleMouseUp)
    },

    handleMouseDown(e) {
      this.mouseDown = true
      const ctx = this.canvasContext
      switch (this.curtool) {
        case 'pencil':
          ctx.lineWidth = this.selectedWidth.value
          ctx.beginPath()
          ctx.moveTo(e.offsetX, e.offsetY)
          break
        case 'text':
          this.startPointX = e.offsetX
          this.startPointY = e.offsetY
          break
        default: return
      }
    },

    handleMouseMove(e) {
      const canvas = this.canvasElement
      const ctx = this.canvasContext
      if(this.mouseDown === true) {
        switch (this.curtool) {
          case 'pencil':
            ctx.lineTo(e.offsetX, e.offsetY)
            ctx.strokeStyle = this.drawcolor
            ctx.stroke()
            break
          case 'text':
            ctx.beginPath()
            ctx.clearRect(0, 0, 1000, 1000)
            if(this.lastCanvasData != null) {
              ctx.putImageData(this.lastCanvasData, 0, 0)
            }
            ctx.shadowColor = '#FFFF00'
            ctx.shadowBlur = 5
            ctx.lineJoin = 'bevel'
            ctx.lineWidth = 2
            ctx.strokeStyle = '#000'
            ctx.strokeRect(this.startPointX, this.startPointY, e.offsetX - this.startPointX, e.offsetY - this.startPointY)
            ctx.closePath()
            ctx.shadowColor = 'transparent'
            ctx.shadowBlur = 0
            ctx.lineJoin = 'miter'
            break
          default: return
        }
      }
    },

    handleMouseUp(e) {
      const canvas = this.canvasElement
      const ctx = this.canvasContext
      if (this.curtool === 'text' && e.offsetX - this.startPointX >= 20 && e.offsetY - this.startPointY >= 20) {
        let input = document.createElement('input')
        let canvasArea = document.getElementsByClassName('canvas-area')[0]
        canvasArea.appendChild(input)
        let textvalue = ''
        input.style.position = "absolute"
        input.style.left = `${this.startPointX + 5}px`
        input.style.top = `${this.startPointY + 5}px`
        input.style.width = `${e.offsetX - this.startPointX - 10}px`
        input.style.height = `${e.offsetY - this.startPointY - 10}px`
        input.style.fontSize =`18px`
        input.style.backgroundColor = 'transparent'
        input.style.border = 'none'
        input.style.outline = 'none'
        input.addEventListener('input', (e) => {
          textvalue = e.target.value
        })
        input.addEventListener('keydown', (e) => {
          if(e.key === 'Enter') {
            input.blur()
          }
        })
        input.addEventListener('blur', () => {
          ctx.clearRect(0, 0, 1000, 1000)
          if(this.lastCanvasData != null) {
            ctx.putImageData(this.lastCanvasData, 0, 0)
          }
          canvasArea.removeChild(input)
          ctx.font=`18px sans-serif`
          ctx.fillText(textvalue, parseInt(input.style.left + 5), parseInt(input.style.top) + parseInt(input.style.height) / 2 + 5)
          this.saveLastData()
        })
        input.focus()
      }
      else {
        this.saveLastData()
      }
      this.mouseDown = false
    },

    // 保存画布图片到本地
    saveCanvasImg() {
      const canvas = this.canvasElement
      const a = document.createElement("a")
      a.href = canvas.toDataURL()
      a.download = '画笔保存的图片'
      a.click()
      this.resetCanvasImg()
    },
  }
}
</script>

<style lang="less" scoped>
.home {
  height: calc(100% - 8px);
  display: flex;
  .origin-img-wrapper {
    margin-right: 30px;
    img {
      // height: 100%;
      width: 100%;
      object-fit: contain;
    }
  }

  .right {
    flex: 1;
    height: 100%;
    position: relative;
    display: flex;
    .canvas-area {
      position: relative;
      background-color: transparent;
      width: fit-content;
      height: fit-content;
    }
    .tools-wrapper {
      font-size: 22px;
      .tool-item {
        color: blue;
        cursor: pointer;
        width: 100px;
        height: 40px;
        line-height: 36px;
        text-align: center;
        border: 2px solid blue;
        border-radius: 5px;
        margin-bottom: 12px;
      }
      .tool-item-selected {
        color: white;
        background-color: blue;
      }
    }
    .change-width-wrapper {
      font-size: 16px;
      .width-item {
        color: blue;
        cursor: pointer;
        width: 46px;
        height: 46px;
        line-height: 44px;
        text-align: center;
        border: 1px solid blue;
        border-radius: 50%;
        margin-bottom: 20px;
      }
      .width-item-selected {
        color: white;
        background-color: blue;
      }
    }
    .color-selector-wrapper {
      .color-item {
        width: 46px;
        height: 46px;
        border-radius: 50%;
        &:not(:last-child) {
          margin-bottom: 20px;
        }
      }
      .color-item-selected {
        transform: scale(1.2);
        box-shadow: 0px 3px 3px black;
      }
    }
    .save-btn {
      font-size: 22px;
      color: blue;
      cursor: pointer;
      width: 100px;
      height: 40px;
      line-height: 36px;
      text-align: center;
      border: 2px solid blue;
      border-radius: 5px;
      margin-bottom: 12px;
    }
    .disabled-btn {
      color: #fff;
      background-color: rgb(146, 140, 140);
      border: 2px solid rgb(146, 140, 140);
    }
  }
}
</style>