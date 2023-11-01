<template>
  <div id="canvas-container">
    <div class="control">
      <a href="javascript:;" class="upload">
        <input
          id="selectImage"
          ref="file"
          type="file"
          accept="image/*"
          @change="uploadImage"
        />上传图片
      </a>
      <button @click="clearCanvas">清除画布</button>
      <button @click="back" :disabled="history.length <= 1">上一步</button>
      <button @click="next" :disabled="future.length === 0">下一步</button>
      <button @click="uploadDrawImage">绘制图片</button>
    </div>
    <div class="box-row">
      <div class="control-row">
        <span>画笔颜色：</span>
        <div
          v-for="(item, key) of colorList"
          :key="key"
          class="color-item"
          :style="{ backgroundColor: item }"
          :class="
            selectedColor === item && selectedModel === 'color' ? 'active' : ''
          "
          @click="changeSelectColor(item)"
        ></div>
      </div>
    </div>
    <div class="box-row">
      <div class="control-row">
        <span>画笔粗细：</span>
        <input
          type="range"
          class="range"
          v-model="lineWidth"
          min="1"
          max="4"
          value="1"
        />
      </div>
    </div>
    <div class="box-row">
      <div class="control-row">
        <span>橡皮擦：</span>
        <img
          src="../assets/svg/erasure.svg"
          class="icon"
          @click="selectErasure"
        />
      </div>
    </div>
    <canvas
      width="600px"
      height="300px"
      ref="canvas"
      @mousedown="startDrawing"
      @mousemove="draw"
      @mouseup="stopDrawing"
      @mouseleave="leaveDrawing"
      :class="selectedModel === 'color' ? 'brush' : 'erasure'"
    ></canvas>
    <ErrorInfo :show="show" :message="message" @changeShow="changeShow" />
  </div>
</template>

<script>
import ErrorInfo from "./ErrorInfo.vue";
export default {
  components: { ErrorInfo },
  data() {
    return {
      ctx: null,
      drawing: false,
      selectedColor: "#ffffff",
      selectedModel: "color",
      lineWidth: 1,
      show: false,
      message: "",
      timer: null,
      history: [],
      future: [],
      colorList: [
        "#ffffff",
        "#f5487f",
        "#ffa822",
        "#134e6f",
        "#ff6150",
        "#1ac0c6",
        "#272643",
      ],
    };
  },
  mounted() {
    this.ctx = this.$refs.canvas.getContext("2d");
    // 初始化当前状态
    this.saveState();
  },
  methods: {
    // 上传图片作为绘制背景
    uploadImage(event) {
      const file = event.target.files[0];
      if (!this.checkImageType(file)) {
        this.show = true;
        this.message = "请上传图片文件进行绘制";
        return;
      }
      const reader = new FileReader();
      reader.onload = (e) => {
        const img = new Image();
        img.onload = () => {
          this.$refs.canvas.width = img.width;
          this.$refs.canvas.height = img.height;
          this.ctx.drawImage(img, 0, 0, img.width, img.height);
        };
        img.src = e.target.result;
      };
      reader.readAsDataURL(file);
    },
    // 下载绘制的图片
    uploadDrawImage() {
      if (this.timer) clearTimeout(this.timer);
      this.timer = setTimeout(() => {
        let dataUrl = this.$refs.canvas.toDataURL("image/png");
        let link = document.createElement("a");
        link.href = dataUrl;
        link.download = "draw_image.png";
        link.click();
      }, 500);
    },
    // 选择橡皮擦
    selectErasure() {
      this.selectedModel = "erasure";
    },
    // 选择画笔
    changeSelectColor(index) {
      this.selectedModel = "color";
      this.selectedColor = index;
    },
    // 警告信息提示
    changeShow(value) {
      this.show = value;
    },
    // 开始绘制
    startDrawing(event) {
      this.drawing = true;
      this.draw(event);
    },
    // 绘制
    draw(event) {
      if (!this.drawing) return;
      if (this.selectedModel === "color") {
        this.ctx.globalCompositeOperation = "source-over";
        this.ctx.lineWidth = this.lineWidth;
        this.ctx.lineCap = "round";
        this.ctx.strokeStyle = this.selectedColor;
      } else {
        this.ctx.globalCompositeOperation = "destination-out";
        this.ctx.lineWidth = 20;
      }
      const x = event.clientX - this.$refs.canvas.offsetLeft;
      const y = event.clientY - this.$refs.canvas.offsetTop;
      this.ctx.lineTo(x, y);
      this.ctx.stroke();
      this.ctx.beginPath();
      this.ctx.moveTo(x, y);
    },
    // 停止绘制
    stopDrawing() {
      this.drawing = false;
      this.ctx.beginPath();
      // 停止绘制记录状态
      this.saveState();
    },
    // 离开画板
    leaveDrawing() {
      this.drawing = false;
      this.ctx.beginPath();
    },
    // 记录绘制状态
    saveState() {
      const dataURL = this.$refs.canvas.toDataURL();
      this.history.push(dataURL);
    },
    // 上一步
    back() {
      if (this.history.length <= 1) return;
      const lastState = this.history[this.history.length - 1];
      this.history.pop();
      const prevState = this.history[this.history.length - 1];
      const img = new Image();
      img.onload = () => {
        this.$refs.canvas.width = img.width;
        this.$refs.canvas.height = img.height;
        this.ctx.drawImage(img, 0, 0, img.width, img.height);
      };
      img.src = prevState;
      this.future.unshift(lastState);
    },
    // 下一步
    next() {
      if (this.future.length === 0) return; // 没有更多的状态可供重做
      const nextState = this.future[0]; // 获取下一个状态
      this.future.shift(); // 删除当前状态
      this.history.push(nextState); // 将下一个状态添加到历史记录中
      const img = new Image();
      img.onload = () => {
        this.$refs.canvas.width = img.width;
        this.$refs.canvas.height = img.height;
        this.ctx.drawImage(img, 0, 0, img.width, img.height);
      };
      img.src = nextState;
    },
    // 清除画布
    clearCanvas() {
      this.$refs.file.value = "";
      this.ctx.clearRect(
        0,
        0,
        this.$refs.canvas.width,
        this.$refs.canvas.height
      );
    },

    /**
     * 工具类
     */
    getDrawImage() {
      let dataUrl = this.$refs.canvas.toDataURL("image/png");
      let blob = this.dataURLToBlob(dataUrl);
      let file = new File([blob], "draw.png");
      return file;
    },
    dataURLToBlob(dataURL) {
      let parts = dataURL.split(";base64,");
      let contentType = parts[0].split(":")[1];
      let raw = window.atob(parts[1]);
      let blob = new Blob([raw], { type: contentType });
      return blob;
    },
    checkImageType(file) {
      if (!/\.(jpg|jpeg|png|GIF|JPG|PNG)$/.test(file.name)) {
        return false;
      } else {
        return true;
      }
    },
  },
};
</script>

<style scoped>
#canvas-container {
  padding: 10px;
  font-size: 14px;
}

a {
  text-decoration: none;
}

canvas {
  border: 1px solid #ddd;
}

.brush {
  cursor: url("../assets/svg/brush.svg") 0 16, default;
}

.erasure {
  cursor: url("../assets/svg/erasure.svg"), default;
}

.icon {
  width: 16px;
  height: 16px;
}

.control {
  display: flex;
  align-items: center;
  margin-bottom: 8px;
}

.control > * {
  margin-right: 8px;
}

.upload {
  padding: 4px 10px;
  height: 20px;
  line-height: 20px;
  position: relative;
  cursor: pointer;
  color: #000;
  background: #fafafa;
  border: 1px solid #ddd;
  border-radius: 4px;
  overflow: hidden;
  display: inline-block;
}
.upload input {
  position: absolute;
  right: 0;
  top: 0;
  opacity: 0;
  filter: alpha(opacity=0);
  cursor: pointer;
}

.color {
  height: 30px;
  background: #fafafa;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.color-item {
  width: 16px;
  height: 16px;
  margin-right: 8px;
  border-radius: 50%;
  cursor: pointer;
  border: 1px solid #ddd;
}

.active {
  border: 1px solid #0075ff;
}

button {
  box-sizing: content-box;
  padding: 4px 10px;
  height: 20px;
  line-height: 20px;
  color: #000;
  background: #fafafa;
  border: 1px solid #ddd;
  border-radius: 4px;
  cursor: pointer;
}

button[disabled] {
  color: #a8abb2;
}

.control-row {
  display: flex;
  align-items: center;
}

.box-row {
  margin-bottom: 8px;
}
</style>
