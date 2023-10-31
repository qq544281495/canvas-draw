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
      <button @click="uploadDrawImage">绘制图片</button>
    </div>
    <div class="box-row">
      <label for="selectColor" class="control-row">
        <span>画笔颜色：</span>
        <input
          id="selectColor"
          type="color"
          value="#000000"
          v-model="selectedColor"
          class="color"
        />
      </label>
    </div>
    <div class="box-row">
      <label for="lineWidth" class="control-row">
        <span>画笔粗细：</span>
        <input
          id="lineWidth"
          type="range"
          class="range"
          v-model="lineWidth"
          min="1"
          max="10"
          value="1"
        />
      </label>
    </div>
    <canvas
      width="600px"
      height="300px"
      ref="canvas"
      @mousedown="startDrawing"
      @mousemove="draw"
      @mouseup="stopDrawing"
      @mouseleave="stopDrawing"
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
      selectedColor: "#000000",
      lineWidth: 1,
      show: false,
      message: "",
      timer: null,
    };
  },
  mounted() {
    this.ctx = this.$refs.canvas.getContext("2d");
  },
  methods: {
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
    uploadDrawImage() {
      if (this.timer) clearTimeout(this.timer);
      this.timer = setTimeout(() => {
        let dataUrl = this.$refs.canvas.toDataURL("image/png");
        let link = document.createElement("a");
        link.href = dataUrl;
        link.download = "draw_image.png";
        link.click();
      }, 500);
      // const file = this.getDrawImage();
      // console.log(file);
      // // 创建图片提交表单
      // let formData = new FormData();
      // formData.append("image", file);
    },
    changeShow(value) {
      this.show = value;
    },
    startDrawing(event) {
      this.drawing = true;
      this.draw(event);
    },
    draw(event) {
      if (!this.drawing) return;
      this.ctx.lineWidth = this.lineWidth;
      this.ctx.lineCap = "round";
      this.ctx.strokeStyle = this.selectedColor;
      this.ctx.lineTo(event.offsetX, event.offsetY);
      this.ctx.stroke();
      this.ctx.beginPath();
      this.ctx.moveTo(event.offsetX, event.offsetY);
    },
    stopDrawing() {
      this.drawing = false;
      this.ctx.beginPath();
    },
    clearCanvas() {
      this.$refs.file.value = "";
      this.ctx.clearRect(
        0,
        0,
        this.$refs.canvas.width,
        this.$refs.canvas.height
      );
    },

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

.control-row {
  display: flex;
  align-items: center;
}

.box-row {
  margin-bottom: 8px;
}
</style>
