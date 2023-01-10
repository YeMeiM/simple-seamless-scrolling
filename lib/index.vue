<template>
  <div class="simple-seamless-scrolling">
    <div ref="SSSList" class="simple-seamless-list-group" @touchstart="stopAnimation" @touchmove="moveAnimation"
      @touchend="runingAnimation" :style="`flex-direction:${flexDirection}`">
      <div class="simple-seamless-list-container" :style="`flex-direction:${flexDirection};`">
        <slot :item="item" v-for="item in list">{{ item }}</slot>
        <slot name="holder" v-if="list.length === 0" />
      </div>
      <div class="simple-seamless-list-container" v-if="showCopy && !unlimited"
        :style="`flex-direction:${flexDirection}`">
        <slot :item="item" v-for="item in list">{{ item }}</slot>
      </div>
    </div>
  </div>
</template>

<script>

/**
 * @event changePosition {postion: 当前位置, height: 组件的高度, width: 组件的宽度, innerHeight: 内部列表的总高, innerWidth: 内部列表的总宽}
 * @prop unlimited 无限模式，按照一个方向无限滚动
 * @prop step 速度，越小越慢，越大越快，默认为1
 * @prop list 列表
 * @prop direction 方向 默认 horizontal 水平，可选参数：{horizontal: 水平, vertical: 垂直}
 * @prop touchMove 触摸移动 默认开启，传入false关闭
 */
export default {
  emits: ['changePosition'],
  props: {
    unlimited: {
      type: Boolean,
      default: false,
    },
    step: {
      type: [String, Number],
      default: 1,
    },
    list: {
      type: Array,
      default() {
        return [];
      },
    },
    direction: {
      type: String,
      default: 'horizontal',
    },
    touchMove: {
      type: Boolean,
      default: true,
    },
  },
  data() {
    return {
      scroll: false,
      showCopy: false,
      onTouch: false,
      distance: 0,
      touch: {
        x: -1,
        y: -1,
      },
      position: 0,
      onStart: true,
      lastTime: null,
    };
  },
  methods: {
    stopAnimation(event) {
      if (!this.unlimited && this.touchMove && this.$refs.SSSList) {
        let flag =
          (this.isHorizontal &&
            this.$refs.SSSList.clientWidth >= this.$el.clientWidth) ||
          (!this.isHorizontal &&
            this.$refs.SSSList.clientHeight >= this.$el.clientHeight);
        if (flag) {
          this.touch.x = event.changedTouches[0].clientX;
          this.touch.y = event.changedTouches[0].clientY;
          this.onTouch = true;
        }
      }
    },
    moveAnimation(event) {
      if (this.onTouch && this.isHorizontal) {
        let x = event.changedTouches[0].clientX;
        let moveX = x - this.touch.x;
        this.touch.x = x;
        this.setPostion(moveX);
      } else if (this.onTouch) {
        let y = event.changedTouches[0].clientY;
        let moveY = y - this.touch.y;
        this.touch.y = y;
        this.setPostion(moveY);
      }
    },
    runingAnimation() {
      if (this.onTouch) {
        this.touch.x = -1;
        this.onTouch = false;
      }
    },
    async init() {
      await this.$nextTick();
      if (
        (this.isHorizontal &&
          this.$refs.SSSList.clientWidth >= this.$el.clientWidth) ||
        (!this.isHorizontal &&
          this.$refs.SSSList.clientHeight >= this.$el.clientHeight)
      ) {
        this.showCopy = true;
        this.onStart = true;
        window.requestAnimationFrame(this.drawAnimation);
      } else {
        this.showCopy = false;
        this.onStart = false;
      }
    },
    drawAnimation(timestamp) {
      if (!this.lastTime) {
        this.lastTime = timestamp;
      }
      if (timestamp - this.lastTime >= 5) {
        this.lastTime = timestamp;

        if (!this.onTouch) {
          let step = -this.step;
          this.setPostion(isNaN(step) ? -1 : step);
        }
      }
      if (this.onStart && this.$refs.SSSList) {
        window.requestAnimationFrame(this.drawAnimation);
      }
    },
    setPostion(step) {
      if (!this.$refs.SSSList) {
        return false;
      }
      if (!this.unlimited) {
        this.position += step;
      }
      if (this.unlimited) {
        let temp = this.isHorizontal
          ? this.$refs.SSSList.clientWidth - this.$el.clientWidth
          : this.$refs.SSSList.clientHeight - this.$el.clientHeight;
        if (Math.abs(this.position) < temp && step !== undefined) {
          this.position += step;
        }
      } else if (step === undefined) {
        this.position = 0;
      } else if (step < 0) {
        if (
          this.isHorizontal &&
          Math.abs(this.position) >= this.$refs.SSSList.scrollWidth / 2
        ) {
          this.position = this.$refs.SSSList.scrollWidth / 2 + this.position;
        } else if (
          !this.isHorizontal &&
          Math.abs(this.position) >= this.$refs.SSSList.scrollHeight / 2
        ) {
          this.position = this.$refs.SSSList.scrollHeight / 2 + this.position;
        }
      } else if (step > 0 && this.position >= 0) {
        let temp = this.isHorizontal
          ? this.$refs.SSSList.scrollWidth
          : this.$refs.SSSList.scrollHeight;
        this.position = -(temp / 2) + this.position;
      }
      if (this.isHorizontal) {
        this.$refs.SSSList.style.transform = `translateX(${this.position}px) translateY(0)`;
      } else {
        this.$refs.SSSList.style.transform = `translateX(0px) translateY(${this.position}px)`;
      }
      this.$emit('changePosition', {
        position: this.position,
        height: this.$el.clientHeight,
        width: this.$el.clientWidth,
        innerHeight: this.$refs.SSSList.clientHeight,
        innerWidth: this.$refs.SSSList.clientWidth,
      });
    },
  },
  components: {},
  mounted() {
    this.init();
  },
  computed: {
    flexDirection() {
      return this.isHorizontal ? 'row' : 'column';
    },
    isHorizontal() {
      return !(this.direction === 'vertical');
    },
  },
  unmounted() {
    this.onStart = false;
  },
  watch: {
    list: "init",
    direction: "init",
  },
}
</script>

<style scoped>
.simple-seamless-scrolling {
  overflow: hidden;
}

.simple-seamless-list-group {
  display: flex;
  width: fit-content;
  white-space: nowrap;
}

.simple-seamless-list-container {
  display: flex;
}
</style>