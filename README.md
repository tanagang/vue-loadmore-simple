基于vue开箱即用的全局组件上滑无限滚动加载更多、下拉刷新  
下图gif可能有卡顿，[图片原地址](https://file.40017.cn/tcyp/tz/refresh2.gif)  
```diff
+ 若适用就来个star吧
```

  ![](https://file.40017.cn/tcyp/tz/refresh2.gif)

### 使用方法
首先项目中安装：npm install vue-loadmore-simple -S
```javascript
//main.js
import {LoadMore} from 'vue-loadmore-simple'
Vue.use(LoadMore)
//template
<template>
    <load-more 
        :pageIndex="pageIndex" 
        :pageSize="pageSize" 
        :totalCount="totalCount" 
        :openRefresh="true"
        @refresh="refresh"
        @loadmore="loadmore">

        <div v-for="(item,index) in list" :key="index">
            ...
        </div>
        
    </load-more>
</template>
<script>
export default {
    data() {
        //注意：当含有上图gif中tab切换或者筛选功能时，请初始化this.totalCount=-1;this.pageIndex=1
        return {
            pageIndex: 1,
            pageSize: 10,
            totalCount: -1 //默认给-1
        }
    },
    methods:{
        loadmore(pageIndex){
            //上滑加载更多，pageIndex为下一页页码,
            this.pageIndex = pageIndex
            console.log('加载更多中...')
        },
        refresh(){
            console.log('您下拉刷新了')
        }
    },
}
</script>

```
```diff
- 注意：当含有上图gif中tab切换或者筛选功能时，请初始化this.totalCount=-1;this.pageIndex=1 
```
### 参数如下
  *  :pageIndex：页码 （必选）
  *  :pageSize：页的大小（必选）
  *  :totalCount：总条数（必选）
  *  :tips：当没有数据时提供的文本（默认值：暂无数据）
  *  :tipsSrc：当没有数据时提供的图标（默认自带），必须是在线地址图标
  *  :openRefresh：true，默认false，只有开启以后才能启动下拉刷新* 
 
### 回调函数
  *  @loadmore：下滑到底时的回调，回调中的参数是下一页的页码
  *  @refresh：下拉刷新

***
github链接
[链接名称](https://github.com/tanagag/vue-loadmore-simple)
