<template>
  <div style="position:relative;">
    <div id="loadMore" style="margin-top:-40px">
      <div class="pull-wrap">
        <i class="loadmore-icon" v-if="state==3"></i>
        <i :class="state==1 ? 'pull-arrow': state==2?'pull-arrow pull-toggle':''"></i>
        <span class="pull-text">下拉刷新</span>
      </div>
      <slot></slot>
      <div class="loadmore-tip" id="loadTips" v-if="totalCount > 0 && totalCount >= pageSize">
        <i v-show="loadTips=='正在加载'" class="loadmore-icon"></i>
        {{loadTips}}
      </div>
      <div v-if="hasData" class="no-data-tip">
        <img :src="tipsSrc">
        <p>{{tips}}</p>
      </div>
    </div>
  </div>
</template>
<script>
var clr, clr2,clr3,clr4;
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
    tipsSrc: {
      type: [String],
      default: "https://file.40017.cn/tcyp/tz/no_data.png"
    },
    tips: {
      type: [String],
      default: "暂无数据"
    },
    openRefresh: {
      type: [String, Boolean],
      default: false
    }
  },
  data() {
    return {
      hasData: false,
      isPull: false,//是否达到了松开坑新的临界点
      state: 1,//1:下拉刷新，2：松开更新，3：更新中
      loadTips: "正在加载"
    };
  },
  watch: {
    pageIndex() {
      this.loadMore();
    },
    totalCount:{
      handler(val, oldval) {
        this.hasData = false;
        if (val > 0) {
          this.loadMore();
        } else if (val == 0) {
          this.hasData = true;
        }
      },
      immediate: true
    }
  },
  methods: {
    //上滑动加载更多
    loadMore() {
      var scope = this;
      scope.loadTips = "正在加载";
      let flag = true;
      let pageSize = this.pageSize;
      let pageIndex = this.pageIndex;
      let totalCount = this.totalCount;

      return new Promise(function(resolve, reject) {
        window.onscroll = function() {
          clr = setTimeout(() => {
            let scrollHeight =
              document.documentElement.scrollHeight ||
              document.body.scrollHeight;
            let scrollTop =
              document.documentElement.scrollTop || document.body.scrollTop;
            let clientHeight =
              document.documentElement.clientHeight ||
              document.body.clientHeight;

            if (!document.getElementById("loadTips")) {
              return;
            }
            let rectBottom =
              document.getElementById("loadTips").getBoundingClientRect()
                .bottom | 0;

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
          }, 1000);
        };
      });
    },
    //下拉刷新
    refresh() {
      var _element = document.getElementById("loadMore"),
        _refreshText = document.querySelector(".pull-text"),
        _refreshWrap = document.querySelector(".pull-wrap"),
        _startPos = 0,
        _transitionHeight = 0;

      var scope = this;
      _element.addEventListener("touchstart",function(e) {
          _startPos = e.targetTouches[0].screenY;
          _element.style.transition = "transform 0s";
        },
        { passive: false }//兼容ios等终端
      );

      _element.addEventListener("touchmove",function(e) {
          var h = _element.getBoundingClientRect().top + 40;
          _transitionHeight =((e.targetTouches[0].screenY - _startPos) * 0.3) | 0;
          //_transitionHeight >= 0 即可下拉
          if (h >= 0 && _transitionHeight >= 0) {
            _refreshWrap.style.transform = "rotateX(0deg)"
            _element.style.transform = "translateY(" + _transitionHeight + "px)"
            if (_transitionHeight > 0 && _transitionHeight < 100) {
              scope.state = 1;
              _refreshText.innerText = "下拉刷新";
              scope.isPull = false;
              if (_transitionHeight > 50) {
                scope.state = 2;
                _refreshText.innerText = "松开更新";
                scope.isPull = true;
              }
            }
          }
        },
        { passive: false }//兼容ios等终端
      );

      _element.addEventListener("touchend",function(e) {
          _element.style.transition = "transform 0.5s ease 0.1s";
          _element.style.transform = "translateY(0px)";
          if (scope.isPull) {
            var h = _element.getBoundingClientRect().top + 40;
            _transitionHeight = e.changedTouches[0].screenY - _startPos;
            //_transitionHeight >= 0 即可下拉
            if (h >= 0 && _transitionHeight >= 0) {
              scope.state = 3;
              _refreshText.innerText = "更新中";
              _element.style.transform = "translateY(40px)";
              clr2 = setTimeout(function() {
                scope.$emit("refresh", true);
                _element.style.transform = "translateY(0px)";
                clearTimeout(clr2);
              }, 1500);
              clr3 = setTimeout(function() {
                _refreshWrap.style.transform = "rotateX(90deg)"
                clearTimeout(clr3);
              }, 1800);
            } else {
               scope.state = 1;
               //_element.style.transform = "translateY(0px)";
            }
          }else{
            _element.style.transform = "translateY(0px)";
            clr4 =setTimeout(function(){
              _refreshWrap.style.transform = "rotateX(90deg)"
              clearTimeout(clr4);
            },600)
            
          }
        },
        { passive: false }//兼容ios等终端
      );
    }
  },
  destroyed() {
    clearTimeout(clr);
    clearTimeout(clr2);
    clearTimeout(clr3);
    clearTimeout(clr4);
  },
  mounted() {
    //this.loadMore()
    if (this.openRefresh == true || this.openRefresh == "true") {
      this.refresh();
    }
  }
};
</script>
<style scoped>
.loadmore-tip {
  color: #aaa;
  font-size: 14px;
  text-align: center;
  height: 50px;
  line-height: 50px;
}
.pull-wrap {
  -webkit-transform: rotateX(90deg);
  transform: rotateX(90deg);
  -webkit-transition: all 0.1s ease;
  transition: all 0.1s ease;
  text-align: center;
  width:100%;
}
.no-data-tip {
  text-align: center;
  padding-top: 36%;
}
.no-data-tip p {
  font-size: 16px;
  margin: 10px auto;
  color: #aaa;
  padding: 0;
}
.no-data-tip img {
  display: block;
  margin: 0 auto;
  width: auto;
  max-width: 150px;
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
  color: #aaa;
  height: 40px;
  line-height: 40px;
}
.pull-arrow {
  vertical-align: -6px;
  height: 18px;
  width: 14px;
  display: inline-block;
  background: url("data:image/svg+xml;utf8;base64,PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0iaXNvLTg4NTktMSI/Pg0KPCEtLSBHZW5lcmF0b3I6IEFkb2JlIElsbHVzdHJhdG9yIDE4LjEuMSwgU1ZHIEV4cG9ydCBQbHVnLUluIC4gU1ZHIFZlcnNpb246IDYuMDAgQnVpbGQgMCkgIC0tPg0KPHN2ZyB2ZXJzaW9uPSIxLjEiIGlkPSJDYXBhXzEiIHhtbG5zPSJodHRwOi8vd3d3LnczLm9yZy8yMDAwL3N2ZyIgeG1sbnM6eGxpbms9Imh0dHA6Ly93d3cudzMub3JnLzE5OTkveGxpbmsiIHg9IjBweCIgeT0iMHB4Ig0KCSB2aWV3Qm94PSIwIDAgMjAxLjg5NCAyMDEuODk0IiBzdHlsZT0iZW5hYmxlLWJhY2tncm91bmQ6bmV3IDAgMCAyMDEuODk0IDIwMS44OTQ7IiB4bWw6c3BhY2U9InByZXNlcnZlIj4NCjxnPg0KCTxnPg0KCQk8cG9seWdvbiBzdHlsZT0iZmlsbDojMDEwMDAyOyIgcG9pbnRzPSIzMy43NjgsNjYuMDE5IDM5LjEwOCw3MS4yNjYgOTcuMTcxLDE0LjIwNCA5Ny4xNzEsMjAxLjg5NCAxMDQuNzE5LDIwMS44OTQgDQoJCQkxMDQuNzE5LDE0LjE5NCAxNjIuNzg2LDcxLjI2NiAxNjguMTI1LDY2LjAxOSAxMDAuOTQ3LDAgCQkiLz4NCgk8L2c+DQo8L2c+DQo8Zz4NCjwvZz4NCjxnPg0KPC9nPg0KPGc+DQo8L2c+DQo8Zz4NCjwvZz4NCjxnPg0KPC9nPg0KPGc+DQo8L2c+DQo8Zz4NCjwvZz4NCjxnPg0KPC9nPg0KPGc+DQo8L2c+DQo8Zz4NCjwvZz4NCjxnPg0KPC9nPg0KPGc+DQo8L2c+DQo8Zz4NCjwvZz4NCjxnPg0KPC9nPg0KPGc+DQo8L2c+DQo8L3N2Zz4NCg==")
    no-repeat;
  transition: all 0.3s ease;
}
.pull-toggle {
  vertical-align: -2px;
  transform: rotate(180deg);
}
</style>
