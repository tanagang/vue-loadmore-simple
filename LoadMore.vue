<template>
  <div>
    <slot></slot>
    <div class="loadmore-tip" id="loadTips" v-if="totalCount > 0 && totalCount >= pageSize"><i v-show="loadTips=='正在加载'" class="loadmore-icon"></i>{{loadTips}}</div>
    <div v-if="hasData" class="no-data-tip">
      <img :src="tipsSrc">
      <p>{{tips}}</p>
    </div>
  </div>
</template>
<script>
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
    tipsSrc:{
      type: [String],
      default: "https://file.40017.cn/tcyp/tz/no_data.png"
    },
    tips: {
      type: [String],
      default: "暂无数据"
    },
    callback: {
      type: Function,
      default() {
        return function() {};
      }
    }
  },
  data() {
    return {
      hasData: false,
      loadTips: "正在加载"
    };
  },
  watch: {
    pageIndex() {
      this.pullDown();
    },
    totalCount(val, oldval) {
      this.hasData = false;
      if (val > 0) {
        this.pullDown();
      } else if (val == 0) {
        this.hasData = true;
      }
    }
  },
  methods: {
    pullDown() {
      var scope = this;
      this.loadTips = '正在加载'
      let flag = true;
      let pageSize = this.pageSize;
      let pageIndex = this.pageIndex;
      let totalCount = this.totalCount;

      return new Promise(function(resolve, reject) {
        window.onscroll = function() {
          setTimeout(() => {
            let scrollHeight = document.documentElement.scrollHeight || document.body.scrollHeight;
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
                  scope.callback(pageIndex + 1);
                  resolve(true);
                } else {
                  scope.loadTips = "没有更多数据了";
                  resolve(false);
                }
              }
            }
          }, 1000);
        }
      });
    }
  },
  mounted() {
    //this.pullDown()
  }
};
</script>
<style scoped>
.loadmore-tip{
  color:#aaa;
  font-size:14px;
  text-align: center;
  height:50px;
  line-height: 50px;
}
.no-data-tip{
    text-align: center;
    padding-top: 36%;
}
.no-data-tip p{
  font-size:14px;
  margin:10px auto;
  color:#aaa;
  padding:0;
}
.no-data-tip img{
  display: block;
  margin:0 auto;
  width:auto;
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
.loadmore-icon{
  vertical-align: -1px;
  width: 12px;
  height: 12px;
  display: inline-block;
  margin:0 5px;
  background: url("data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAFAAAABQCAMAAAC5zwKfAAAABGdBTUEAALGPC/xhBQAAAAFzUkdCAK7OHOkAAAGzUExURUxpcaSmo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo6Smo7OajWMAAACQdFJOUwDzVSjxAgf0ChUBCd/hpyn6+ai70Qz1uB92XuuOR5xNbnBc0ulZd4eNIdsW/myUA1iBhMgnbQiWb7zsJM/l1hqqFEjOqQ3GrbH4LGQrr/CK49NdjIncrLNiaRtbtRl1771FUHjQj0aQBt5axWCTHeRmt57dnbTyg6vV7eIgEk4mUdcwOvceDgQRiPylmZgL2vNJv00AAAM1SURBVFjDrZl3WxpBEMaPQ3ovKiAKSreABREVLLEbTewaY4mJJb333nvhIwd2jqNzbeYvHnb2d+zd7Du3LxRVPyaPeqK2Mb8sY3n5yG6L9hxNUuJD7Tk57s6URffxiUcthqZoNaoyNUJlbFUIxDW26zN1Q9/eKACnO2jJcEaLU8eX12sontiUmu5P7tD0TrJ/OtVUPGLo5YVzPChMaZhPLmqLB7WL3vmGwnizg5vXweab9+iqi9LRe2b2ih0cuMRj9km6d2un7brZCniYqMfrmmXSLBua+lfWbFiY1Nmu2lmRTSZpOM59b+LDTPJmpFaKKQ0ZMhe/anDJID9tqj4+ugLjq3K+9SVfhRkro1XvSieMWgXsVJ0R5nRWuePXB2AsrBWyRbXrMGtAWTHUBiN9QkWkD+a1VdQzfD8kXJaGYGZZhTtgf8zFhANjW7BnSndhM/nSJ0o51T7Y1yX6AvUnF6fscqjHQNE1QK9clMhwgZoV1ueE/Sa++zwhACer90SfF+TigZHPOcJIvisMEv5FCf2RChHEINPfbhCl10gBaog+6qEXthK6m5IU7ixi4hN8JhvcPC4NOH4Y/pKvGfJr5yiJ8Y39dJWs+C2FFn9yvAtKPOC/HHACj9dFVnweD3iOAE14wGukqhV4QCL9djwedYV0OkTghxwwhAj054BBRODTHPASIpBUzWVEoAUbOIK9ZD32Q/Fhlw16YaNvPXRxQJcvdIGlOpFbAPUbu0lBG/XgAf+SRr+FuGYrxqtIZeG48YAKog+qSTwiwgtnmQVCNHEhjkeEl3YbHpA5VqzhEQOSDj7VAo5mdjUa8DUcHp/H0Ihroo+3dVuBiAN4zVAyFsG6FouYNzGMCSzi6D0g+tCqx/RKmBHEHXdfMO6T7R2PIy2fTvnmVt5MC3GZaT8sqe98nvUd1u4L79dO2w9n+8bPX1/5LPv9IWtIbtNVe6GS3mYMyQNeN9LRXGSZLnuXSi3TJe9ywTJNf+SpPSWmrmrK+sx7m755P9hvnSqxtg0Bvk9b5+RjOwd1AgpohtsYnxHaC3Gte+gMnjN75Z8L9jOPFB3WnJK/P7I/VuUfs0V7TrncmP8jtvO4FdRBjgAAAABJRU5ErkJggg==")
    no-repeat;
  background-size: 12px 12px;
  -webkit-animation: rotate-loading 0.6s linear forwards infinite;
  animation: rotate-loading 0.6s linear forwards infinite;
}
</style>