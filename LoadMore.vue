<template>
  <div id="loadMore" class="loadmore" @touchstart="touchstart($event)" @touchmove="touchmove($event)" @touchend="touchend($event)">
    <div class="pull-wrap">
        <div v-show="openRefresh">
          <transition  name="fade"><i class="loadmore-icon" v-if="state==3&&refreshTips"></i></transition>
          <transition  name="fade"><i v-show="refreshTips" :class="state==1 ? 'pull-arrow pull-toggle': state==2?'pull-arrow':''"></i></transition>
          <transition  name="fade"><span v-if="refreshTips" class="pull-text">{{refreshTips}}</span></transition>
        </div>
    </div>
    <slot></slot>
    <div class="loadmore-tip" id="loadTips" v-if="totalCount > 0 && totalCount >= pageSize">
      <i v-show="loadTips=='正在加载中'" class="loadmore-icon"></i>
      {{loadTips}}
    </div>
  </div>
</template>
<script>
var clr, clr2 ,clr3;
export default {
  props: {
    pageIndex: {
      type: [Number, String],
      default: "1"
    },
    pageSize: {
      type: [Number, String],
      default: "10"
    },
    totalCount: {
      type: [Number, String],
      default: "0"
    },
    tips: {
      type: [String],
      default: "暂无数据"
    },
    openRefresh: {
      type: [Boolean],
      default: false
    }
  },
  data() {
    return {
      isPull: false,//是否达到了松开刷新的临界点
      state: 1,//1:下拉刷新，2：松开刷新，3：更新中
      refreshTips:'',
      loadTips: "正在加载中"
    };
  },
  watch: {
    pageIndex() {
      this.loadMore();
    },
    totalCount:{
      handler(val, oldval) {
       if (val > 0) {
          this.loadMore();
        }
      },
      immediate: true
    }
  },
  methods: {
    //上滑动加载更多
    loadMore() {
      var scope = this;
      scope.loadTips = "正在加载中";
      let flag = true;
      let pageSize = this.pageSize;
      let pageIndex = this.pageIndex;
      let totalCount = this.totalCount;
      // console.log(pageSize,pageIndex,totalCount)
      return new Promise(function(resolve, reject) {
        window.onscroll = function() {
          clr = setTimeout(() => {
            let scrollHeight =  document.documentElement.scrollHeight || document.body.scrollHeight;
            let scrollTop = document.documentElement.scrollTop || document.body.scrollTop;
            let clientHeight = document.documentElement.clientHeight || document.body.clientHeight;
            if (!document.getElementById("loadTips")) {
              return;
            }
            let rectBottom = document.getElementById("loadTips").getBoundingClientRect().bottom | 0;
            //if (scrollTop + clientHeight >= scrollHeight - 25) {
            if (rectBottom < clientHeight + 25) {
              if (flag) {
                flag = false;
                var size = totalCount - pageIndex * pageSize;
                if (size > 0) {
                  scope.$emit("loadmore", pageIndex + 1);
                  resolve(true);
                } else {
                  scope.loadTips = "没有更多数据了";
                  resolve(false);
                }
                clearTimeout(clr);
              }
            }
          }, 400);
        };
      });
    },
    //下拉刷新
    refresh() {
        this._startPos = 0
        this._transitionHeight = 0
        this.obj = document.getElementById("loadMore")
        this.objTop = this.obj.offsetTop
    },
    touchstart(event){
      if (!this.openRefresh) {
        return;
      }
      this._startPos = event.targetTouches[0].screenY
      this.obj.style.transition = "transform 0s linear"
    },
    touchmove(event){
      if (!this.openRefresh) {
        return;
      }
      var h = Math.ceil(this.obj.getBoundingClientRect().top)
      this._transitionHeight =((event.targetTouches[0].screenY - this._startPos) * 0.4) | 0
      if (h>=this.objTop&&this._transitionHeight >= 0) {
        if (typeof event.cancelable !== 'boolean' || event.cancelable) {
          event.preventDefault();
        }
        this.obj.style.transform = "translateY(" + this._transitionHeight + "px)"
        if (this._transitionHeight > 0 && this._transitionHeight < 100) {
          this.state = 1;
          this.refreshTips = "下拉刷新";
          this.isPull = false;
          if (this._transitionHeight > 50) {
            this.state = 2;
            this.refreshTips = "松开更新";
            this.isPull = true;
          }
        }
      }
    },
    touchend(event){
      if (!this.openRefresh) {
        return;
      }
      this.obj.style.cssText = `transition:transform 0.2s ease-out;transform:translateY(0);`
      if (this.isPull) {
        var h = Math.ceil(this.obj.getBoundingClientRect().top + 40)
        this._transitionHeight = event.changedTouches[0].screenY - this._startPos;
        if (h>=this.objTop&&this._transitionHeight >= 0) {
          this.state = 3;
          this.refreshTips = "正在刷新中";
          this.obj.style.transform = "translateY(40px)";
          clr2 = setTimeout(()=> {
            this.obj.style.transform = "translateY(0)";
            clearTimeout(clr2);
          }, 1000);
          clr3 = setTimeout(()=> {
            this.refreshTips=''
            this.isPull = false
            this.$emit("refresh", true);
          }, 1500);
        } else {
            this.state = 1;
            this.obj.style.transform = "translateY(0)"
        }
      }else{
        clr2 = setTimeout(()=> {
          this.state = 1;
          this.refreshTips = ''
          clearTimeout(clr2);
        }, 400);
      }
    }
  },
  destroyed() {
    clearTimeout(clr);
    clearTimeout(clr2);
    clearTimeout(clr3);
  },
  mounted() {
    //this.loadMore()
    if (this.openRefresh) {
      this.$nextTick(()=>{
        this.refresh()
      })
    }
  }
};
</script>
<style scoped>
.loadmore{
  margin-top:-40px;
}
.fade-enter-active,.fade-leave-active {
  -webkit-transition: all 0.2s ease;
  transition: all 0.2s ease;
}
.fade-enter, .fade-leave-to {
  opacity: 0;
}
.loadmore-tip {
  color: #999;
  font-size: 14px;
  text-align: center;
  height: 50px;
  line-height: 50px;
}
.pull-wrap {
  height:40px;
  line-height: 40px;
  text-align: center;
  width:100%;
}

@keyframes rotate-loading {
  0% {
    -webkit-transform: rotate(0);
    transform: rotate(0);
  }
  100% {
    -webkit-transform: rotate(360deg);
    transform: rotate(360deg);
  }
}
@-webkit-keyframes rotate-loading {
  0% {
    -webkit-transform: rotate(0);
    transform: rotate(0);
  }
  100% {
    -webkit-transform: rotate(360deg);
    transform: rotate(360deg);
  }
}
.loadmore-icon {
  vertical-align: -1px;
  width: 12px;
  height: 12px;
  display: inline-block;
  background: url("data:image/png;base64,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")
    no-repeat;
  background-size: 12px 12px;
  -webkit-animation: rotate-loading 0.6s linear forwards infinite;
  animation: rotate-loading 0.6s linear forwards infinite;
}
.pull-text {
  text-align: center;
  color: #999;
  height: 40px;
  font-size:14px;
  line-height: 40px;
}
.pull-arrow {
  vertical-align: -4px;
  height: 18px;
  width: 14px;
  display: inline-block;
  background: url("data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADcAAABMCAYAAADJPi9EAAAAGXRFWHRTb2Z0d2FyZQBBZG9iZSBJbWFnZVJlYWR5ccllPAAAAyJpVFh0WE1MOmNvbS5hZG9iZS54bXAAAAAAADw/eHBhY2tldCBiZWdpbj0i77u/IiBpZD0iVzVNME1wQ2VoaUh6cmVTek5UY3prYzlkIj8+IDx4OnhtcG1ldGEgeG1sbnM6eD0iYWRvYmU6bnM6bWV0YS8iIHg6eG1wdGs9IkFkb2JlIFhNUCBDb3JlIDUuMy1jMDExIDY2LjE0NTY2MSwgMjAxMi8wMi8wNi0xNDo1NjoyNyAgICAgICAgIj4gPHJkZjpSREYgeG1sbnM6cmRmPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5LzAyLzIyLXJkZi1zeW50YXgtbnMjIj4gPHJkZjpEZXNjcmlwdGlvbiByZGY6YWJvdXQ9IiIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bWxuczp4bXBNTT0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL21tLyIgeG1sbnM6c3RSZWY9Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC9zVHlwZS9SZXNvdXJjZVJlZiMiIHhtcDpDcmVhdG9yVG9vbD0iQWRvYmUgUGhvdG9zaG9wIENTNiAoV2luZG93cykiIHhtcE1NOkluc3RhbmNlSUQ9InhtcC5paWQ6QzIwQ0ZDMDcwRjUwMTFFQThDMEVGMTg1QzRDMDZEREIiIHhtcE1NOkRvY3VtZW50SUQ9InhtcC5kaWQ6QzIwQ0ZDMDgwRjUwMTFFQThDMEVGMTg1QzRDMDZEREIiPiA8eG1wTU06RGVyaXZlZEZyb20gc3RSZWY6aW5zdGFuY2VJRD0ieG1wLmlpZDpDMjA4REQ1MDBGNTAxMUVBOEMwRUYxODVDNEMwNkREQiIgc3RSZWY6ZG9jdW1lbnRJRD0ieG1wLmRpZDpDMjA4REQ1MTBGNTAxMUVBOEMwRUYxODVDNEMwNkREQiIvPiA8L3JkZjpEZXNjcmlwdGlvbj4gPC9yZGY6UkRGPiA8L3g6eG1wbWV0YT4gPD94cGFja2V0IGVuZD0iciI/Ps/hVj8AAAR4SURBVHja7JtNTxNBGMd3l/Ia44ELXPTm1e+B0SqGRAySaKAx8eTNxEQvHjRe0As02AQFNHogoph4Np6I38H4AYQSXspLW+r/aXaaYZjZmW13y26dSZ60u8zszm//8zwz83Rxa7Wa06nFczq4WDgLl8CSEU/k83mHgozruqEuRG344ETt+Wvgb1dxfM8/fIvva6wdX46Pjxvt2N88zzO6fyaTcXK5nBoupnIfHc5zxzdhD2BzbVUuDjAaEJInPet/nUurz0nBuDLr10kdXE4D1nDzuABjgYN/TcHmQzSJBTBSOD/KElihieaRA3oxKFZo4RJ5tJ9OHBw6lcN8pAKrigqjblk2l+LcG7pWkuDuwuYV0C9hT07cFJPy0dHR02q1+lyxWKBrTScBjsAWFGDfYY+g1F/+fFdXl7O3t1eEPWYrFYWCU2cJpwQD0Dd07oq/hLooqXKhVCo5lUolizoqwEIrgF4cYFgfrmHIXePWmxWxDtSrHB4e0vCkwyzafFUB+vdqG1yQYgSWNQhA9U9Sz38A12GriuoLzQA2AzcZBEbDTFzVqwr53sHBgUMK0neU0SgBw8LdgS3qwPxQb7xtIvU4NSMD9EKCLZkoRp8EZ5J8kqgXGaDXKhgFD3EohlGNPQxMC+JpHeBkFHATAYp9AVyWBwmjGq8eDc39/f36btoQcNF/6E3D3YYtK/62CpAbkmVV6BQFK7u7u7LTQYBLQYCeBuyDCsy/6anhFVa1RkoAiinUaxrQiwqsVdU06jHAzypA3H/CBG4EHVSCAWBUsY8LUs2NQD2WWFpRNF/GvUd0cLOK4CFVjF/pt6qagXrUjzHYiuz+CG4zOrjzMjDYaFAo1/haTaxP+clyuUwL5xNGZXt729nZ2eHnvVOACgXPBab20PAZbj7DdeQTPsaDggRTLaDOCUkJor+/3xkaGjqlNh1jber09PToUhpjuO9HfN7iHtgLHdwrNKJxcRmVf6HRknaybDJCqvIwpBhbsfjDTVqvu7t7HA9iHQ/rEuqso6/vAuH8UjDxHxYhTfosBg4aehsbG9RBR7HyqQ/bwcFB3epmhvpBaovXainjLP4eEEYdeijUGUVUrBfyu4GBgfoQZv6o20LFnreMMOkUGDnbmtqLuhjMe+mFE+e9sC6QeDimHplq3ku1crpVS6rhSD3yu7C+l6rfxEm9MD9pG8Ox5ZVstdDuyGnqe6l7m4FyLabqhYKLakvTqu8JmbLOUY73vUjgml1Dxq2eLnKm9g0ik3kvlXB8npN2FqohqoUTX3tKym6BbHNzs74VUgUXL8nRMaiQYjQ0t7a2lBveTFp9zk8zNBJJtKkVswKZoMZJLyyLViwW6ykJLVyUyZ52FNZfUlDsd2qHpaigLKjY14AtnIWzcBbOwlk4C2fhLJyF+9/h2rXlcSWbYZc2mnHuHdsF1yvZZPaytxXSDvebP6CMVV9f35/h4eGO8Ln3sB/c8U/YIkvRRWlnoRy9xDwF/3pI/oeh+BrfS3Hnalz7T/AWLnnlnwADAJpuWXrR2Wc8AAAAAElFTkSuQmCC")
    no-repeat;
  background-size:14px 18px;
  -webkit-transition: all 0.3s ease;
  transition: all 0.3s ease;
}
.pull-toggle {
  -webkit-transform: rotate(180deg);
  transform: rotate(180deg);
}
</style>
