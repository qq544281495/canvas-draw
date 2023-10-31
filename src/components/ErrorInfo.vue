<template>
  <div id="error-info" v-show="show">
    <div class="message">
      <img src="../assets/svg/warning.svg" class="icon" />
      <span>{{ message }}</span>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    show: {
      type: Boolean,
      require: true,
    },
    message: {
      type: String,
      require: true,
    },
  },
  data() {
    return {
      timer: null,
    };
  },
  watch: {
    show: {
      handler(value) {
        if (value) {
          this.timeoutHide();
        }
      },
    },
  },
  methods: {
    timeoutHide() {
      this.timer = setTimeout(() => {
        this.$emit("changeShow", false);
      }, 2000);
    },
  },
  beforeDestroy() {
    clearTimeout(this.timer);
  },
};
</script>

<style lang="css" scoped>
#error-info {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
}

.message {
  display: flex;
  align-items: center;
  position: absolute;
  top: 20px;
  left: 50%;
  transform: translateX(-50%);
  padding: 6px 18px;
  border-radius: 4px;
  background-color: #fdf6ec;
  color: #e6a23c;
}

.icon {
  width: 16px;
  height: 16px;
  margin-right: 6px;
}
</style>
