<template>
  <div>
    <div class="slider-remove" ref="slider">
      <div class="slider-content" ref="sliderContent" :style="swipeStyle" @touchstart="touchStart" @touchmove="touchMove" @touchend="touchEnd">
        <slot></slot>
      </div>
      <div class="btn-remove slider-flex" :style="defaultClass+swipeStyle" ref="remove">
        <div class="slider-flex" @click.stop="nextEvent" v-if="btnTwo" :style="{background:backgroundTwo,color:colorTwo}">{{this.btnTwo}}</div>
        <div class="slider-flex" @click.stop="remove" :style="{background:background,color:color}">{{this.btn}}</div>
      </div>
    </div>
  </div>
</template>
<script>
export default {
  props: {
    background: {
      type: [Number, String],
      default: "red"
    },
    backgroundTwo: {
      type: [Number, String],
      default: "#ccc"
    },
    color: {
      type: [Number, String],
      default: "#fff"
    },
    colorTwo: {
      type: [Number, String],
      default: "#fff"
    },
    btn: {
      type: [Number, String],
      default: "删除"
    },
    btnTwo: {
      type: [Number, String],
      default: ""
    },
    fontSize: {
      type: [Number, String],
      default: "14px"
    },
    width: {
      type: [Number, String],
      default: "80"
    },
    params: {
      type: [Number, String, Object],
      default: ""
    }
  },
  data() {
    return {
      startX: 0, //触摸位置
      startY: 0, //触摸位置
      endX: 0, //结束位置
      moveX: 0, //滑动时的位置
      moveY: 0, //滑动时的位置
      disX: 0, //移动距离
      timeOutEvent: null,
      swipeStyle: "" //滑动时的效果,使用v-bind:style="swipeStyle"
    };
  },
  computed: {
    defaultClass() {
      if (this.btnTwo) {
        return `font-size:${this.fontSize};width:${this.width *
          2}px;right:-${this.width * 2}px;`;
      } else {
        return `font-size:${this.fontSize};width:${this.width}px;right:-${
          this.width
        }px;`;
      }
    }
  },
  mounted() {
    this.wd = this.$refs.remove.offsetWidth;
    //获取元素下第一个子节点
    var firstChild = this.$refs.sliderContent.firstElementChild;
    var remove = this.$refs.remove;
    //兼容IE和火狐谷歌等的写法(获取元素属性)
    if (window.getComputedStyle) {
      var computedStyle = getComputedStyle(firstChild, null);
    } else {
      computedStyle = firstChild.currentStyle; //兼容IE的写法
    }
    //动态更改remove按钮的margin
    remove.style.height = computedStyle.height;
    remove.style.marginTop = computedStyle.marginTop;
    remove.style.marginBottom = computedStyle.marginBottom;  
  },
  methods: {
    //返回角度
    GetSlideAngle(dx, dy) {
      return Math.atan2(dy, dx) * 180 / Math.PI;
    },
    //根据起点和终点返回方向 1：向上，2：向下，3：向左，4：向右,0：未滑动
    GetSlideDirection(startX, startY, moveX, moveY) {
      var dy = startY - moveY;
      var dx = moveX - startX;
      var result = 0;
      //如果滑动距离太短
      // if (Math.abs(dx) < 20 && Math.abs(dy) < 20) {
      //     return result;
      // }
      var angle = this.GetSlideAngle(dx, dy);
      if (angle >= -45 && angle < 45) {
        result = "swiperight";
      } else if (angle >= 45 && angle < 135) {
        result = "swipeup";
      } else if (angle >= -135 && angle < -45) {
        result = "swipedown";
      } else if (
        (angle >= 135 && angle <= 180) ||
        (angle >= -180 && angle < -135)
      ) {
        result = "swipeleft";
      }
      return result;
    },
    touchStart(ev) {
      ev = ev || event;
      //tounches类数组，等于1时表示此时有只有一只手指在触摸屏幕
      if (ev.touches.length == 1) {
        this.timeOutEvent = setTimeout(() => {
          //此处为长按事件
          this.$emit("longpress", this.params);
          return;
        }, 700);
        // 记录开始位置
        this.startX = ev.targetTouches[0].clientX;
        this.startY = ev.targetTouches[0].clientY;

        this.backInitX()
      }
    },
    //先收缩所有回归初始状态后，再滑出当前按钮
    backInitX(){
      let slider = document.querySelectorAll(".slider-remove");
  
      for (var i = 0; i < slider.length; i++) {
        if (slider[i] != this.$refs.slider) {
          slider[i].children[0].style.transform = "translateX(0)";
          slider[i].children[1].style.transform = "translateX(0)";
        }
      }
    },
    touchMove(ev) {
      ev = ev || event;
      clearTimeout(this.timeOutEvent);
      //获取删除按钮的宽度，此宽度为滑块左滑的最大距离

      if (ev.touches.length == 1) {
        // 滑动时距离浏览器左侧实时距离
        this.moveX = ev.targetTouches[0].clientX;
        this.moveY = ev.targetTouches[0].clientY;

        this.direction = this.GetSlideDirection(
          this.startX,
          this.startY,
          this.moveX,
          this.moveY
        );

        if (this.direction !== "swipeleft" && this.direction !== "swiperight") {
          return;
        }

        //起始位置减去 实时的滑动的距离，得到手指实时偏移距离
        this.disX = this.startX - this.moveX;
        // 如果是向右滑动或者不滑动，不改变滑块的位置
        if (this.disX < 0 || this.disX == 0) {
          this.swipeStyle = `transform:translateX(0px);`;
          // 大于0，表示左滑了，此时滑块开始滑动
        } else if (this.disX > 0) {
          //具体滑动距离我取的是 手指偏移距离*2。
          this.swipeStyle = `transform:translateX(-${this.disX}px);`;
          // 最大也只能等于删除按钮宽度
          if (this.disX >= this.wd) {
            this.swipeStyle = `transform:translateX(-${this.wd}px);`;
          }
        }
      }
    },
    touchEnd(ev) {
      ev = ev || event;
      clearTimeout(this.timeOutEvent);
      if (ev.changedTouches.length == 1) {
        let endX = ev.changedTouches[0].clientX;
        this.disX = this.startX - endX;
        //如果距离小于删除按钮一半,强行回到起点
        if (this.disX < this.wd / 2) {
          this.swipeStyle = `transform:translateX(0);`;
        } else {
          //大于一半 滑动到最大值
          this.swipeStyle = `transform:translateX(-${this.wd}px);`;
        }
      }
    },
    nextEvent() {
      this.$emit("nextEvent");
    },
    remove() {
      this.$emit("callback");
    }
  }
};
</script>
<style scoped>
.slider-remove {
  width: 100%;
  position: relative;
  user-select: none;
  flex-wrap: nowrap;
  z-index: 1;
  overflow: hidden;
}

.slider-content {
  width: 100%;
  z-index: 100;
  overflow: hidden;
  transition: all 0.2s ease-out;
}

.btn-remove {
  position: absolute;
  top: 0;
  height: 100%;
  text-align: center;
  font-size: 16px;
  transition: all 0.2s ease-out;
}
.btn-remove > div {
  width: 100%;
  height: 100%;
}
.slider-flex {
  -webkit-box-align: center;
  align-items: center;
  -webkit-box-orient: horizontal;
  -webkit-box-direction: normal;
  flex-direction: row;
  -webkit-box-pack: justify;
  justify-content: center;
  display: -webkit-box;
  display: flex;
  box-sizing: border-box;
}
</style>