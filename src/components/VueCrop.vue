<template>
  <div class="dialog-cover" v-if="show">
    <div v-if="upload" class="dialog">
      <div class="close" @click="closeUpload()">×</div>
      <input type="file" id="file" name="file" ref="file" @change="fileChange()" accept="image/png,image/jpeg,image/gif,image/jpg"
      style="visibility:hidden;position:absolute;"
      >
      <label class="upload" for="file">点击上传</label>
    </div>
    <div v-if="cropDialog" class="dialog">
      <div class="close" @click="closeCrop()">×</div>
      <h3 style="text-align: center;color:#444444;margin-top:20px;">选取图片区域</h3>
      <div class="container">
        <canvas id="canvas" ref="canvas" class="canvas" width="300" height="300"
        @mousedown="canvasDown($event)" @mousemove="canvasMove($event)" @mouseup="canvasUp($event)"
        >Not Support Canvas</canvas>
      </div>
      <div class="tool">
        <div>-</div>
        <input type="range" v-model="scale" id="range" class="range" min="0.1" max="2.0" step="0.01" value="1"
        v-on:mousedown="rangeDown($event)" v-on:mousemove="rangeMove($event)" v-on:mouseup="rangeUp($event)">
        <div>+</div>
      </div>
      <button
      ref="button"
        @click="cropImage()"
        style="margin:30px auto 20px;padding:6px 10px;font-size:14px;width:100px;background:#ffffff;border:1px solid #999999;color:#444444;outline:none;cursor:pointer;border-radius:4px;">
          确 认</button>
    </div>
  </div>
</template>
<script>
export default {
  name: 'vue-crop',
  props: {
    visible: {type: Boolean, default: false}
  },
  data: function () {
    return {
      url: '',
      canvas: '',
      context: '',
      scale: 1,
      file: '',
      shape: {x: 0, y: 0},
      offset: {x: 0, y: 0},
      mouse: {x: 0, y: 0},
      image: new Image(),
      show: false,
      isRangeDown: false,
      isDropDown: false,
      upload: true,
      cropDialog: false
    }
  },
  created () {
  },
  methods: {
    closeUpload () {
      this.upload = true
      this.cropDialog = false
      this.show = false
      this.$emit('on-close')
    },
    closeCrop () {
      this.upload = true
      this.cropDialog = false
    },
    fileChange () {
      this.shape = {x: 0, y: 0}
      this.offset = {x: 0, y: 0}
      this.mouse = {x: 0, y: 0}
      this.scale = 1
      let inputDOM = this.$refs['file']
      this.file = inputDOM.files[0]
      const reader = new FileReader()
      reader.readAsDataURL(this.file) // 转化成base64数据类型
      const that = this
      reader.onload = function (e) {
        that.url = e.target.result
        that.upload = false
        that.cropDialog = true
      }
    },
    rangeDown () {
      this.isRangeDown = true
    },
    rangeMove () {
      if (this.isRangeDown) {
        this.drawImageByScale(this.scale)
      }
    },
    rangeUp () {
      this.isRangeDown = false
    },
    canvasDown (e) {
      e.preventDefault()
      this.isDropDown = true
      this.mouse = this.windowToCanvas(e.clientX, e.clientY) // 鼠标按下时位置
    },
    canvasMove (e) {
      e.preventDefault()
      if (this.isDropDown && this.isPointInPath()) {
        let _mouse = this.windowToCanvas(e.clientX, e.clientY) // 鼠标按下并移动中的位置
        this.offset = { // 鼠标的位移
          x: _mouse.x - this.mouse.x,
          y: _mouse.y - this.mouse.y
        }
        this.mouse.x = _mouse.x
        this.mouse.y = _mouse.y
        this.context.clearRect(0, 0, this.canvas.width, this.canvas.height)
        this.shape.x = this.shape.x + this.offset.x
        this.shape.y = this.shape.y + this.offset.y
        this.drawImageByDrop(this.shape.x, this.shape.y)
      }
    },
    canvasUp (e) {
      e.preventDefault()
      this.isDropDown = false
    },
    windowToCanvas (x, y) {
      const bound = this.canvas.getBoundingClientRect()
      return { x: x - bound.left, y: y - bound.top }
    },
    drawImageByScale (scale) {
      this.canvas = document.getElementById('canvas')
      if (!this.canvas) return
      const imgWidth = this.image.width * scale
      const imgHeight = this.image.height * scale
      this.context = this.canvas.getContext('2d')
      this.context.clearRect(0, 0, this.canvas.width, this.canvas.height)
      this.context.drawImage(this.image, this.shape.x, this.shape.y, imgWidth, imgHeight)
    },
    drawImageByDrop (dx, dy) {
      const imgWidth = this.image.width * this.scale
      const imgHeight = this.image.height * this.scale
      this.context.clearRect(0, 0, this.canvas.width, this.canvas.height)
      this.context.drawImage(this.image, dx, dy, imgWidth, imgHeight)
    },
    cropImage () {
      const img = this.canvas.toDataURL('image/png')
      this.closeUpload()
      const blog = this.dataURLtoFile(img)
      console.log('选中的区域:===:> ', blog)
      this.file = ''
      this.url = ''
      this.$emit('on-success', img)
    },
    isPointInPath () {
      this.context.rect(0, 0, this.canvas.width, this.canvas.height)
      let bool = this.context.isPointInPath(this.mouse.x, this.mouse.y)
      return bool
    },
    dataURLtoFile (dataurl) {
      const arr = dataurl.split(',')
      const mime = arr[0].match(/:(.*?);/)[1]
      const bstr = atob(arr[1])
      let n = bstr.length
      let u8arr = new Uint8Array(n)
      while (n--) {
        u8arr[n] = bstr.charCodeAt(n)
      }
      const dropImgName = new Date().getTime()
      return new File([u8arr], dropImgName, { type: mime })
    }
  },
  watch: {
    url (val) {
      const that = this
      this.image.src = val
      this.image.onload = function () {
        that.drawImageByScale(that.scale, that)
      }
    },
    visible (val) {
      this.show = val
    }
  }
}
</script>
<style scoped>
.upload {
  margin: 60px auto;
  width: 226px;
  display: block;
  background:#EE6363;
  padding: 8px 14px;
  color:#ffffff;
  font-size: 17px;
  border-radius: 3px;
  outline: none;
  cursor: pointer;
}
.dialog-cover {
  position: fixed;
  background:rgba(0,0,0,0.8);
  height: 100%;
  width:100%;
  z-index: 200;
  top: 0;
  left: 0;
}
.close {
  font-size: 18px;
  position: absolute;
  right: 10px;
  cursor: pointer;
}
.dialog {
  width: 600px;
  border-radius: 5px;
  left: 50%;
  top: 43%;
  -moz-transform: translate(-50%, -50%);
  -ms-transform: translate(-50%, -50%);
  -webkit-transform: translate(-50%, -50%);
  transform: translate(-50%, -50%);
  background: #ffffff;
  position: fixed;
  z-index: 300;
}
.container {
  display: flex;
  flex-direction:column;
  margin: 20px auto;
}
.canvas {
  display: block;
  border-radius: 2px;
  border: 2px dotted #777777;
  margin: 20px auto;
  cursor: grab;
}
.tool {
  display: flex;
  width: 302px;
  margin: 0 auto;
}
.range {
  margin: 0px auto;
  display: block;
  cursor: pointer;
  width: 270px;
}
</style>
