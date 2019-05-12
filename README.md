基于vue开箱即用的全局组件上滑加载更多，下拉刷新
### 使用方法
```js
    首先安装：npm install vue-loadmore-simple -S
    //main.js
    import {LoadMore} from 'vue-loadmore-simple'
    Vue.use(LoadMore)
    //template
    <template>
        <load-more :pageIndex="pageIndex" :pageSize="pageSize" :totalCount="totalCount" @loadmore="loadmore" :openRefresh="true"  @refresh="refresh">
            <div>
                <div v-for="(item,index) in list" :key="index">
                    ...
                </div>
            </div>
        </load-more>
    </template>
    <script>
    export default {
        data() {
            return {
                pageIndex: 1,
                pageSize: 10,
                totalCount: 0
            }
        },
        methods:{
            loadmore(pageIndex){
                this.pageIndex = pageIndex
                this.getList();
            },
            refresh(){
                console.log('您下拉刷新了')
            }
        },
    }
    </script>
```
```
    参数如下：
    :pageIndex：页码 （必选）
    :pageSize：页的大小（必选）
    :totalCount：总条数（必选）
    :tips：当没有数据时提供的文本（默认值：暂无数据）
    :tipsSrc：当没有数据时提供的图标（默认自带），必须是在线地址图标
    :openRefresh：true，默认false，只有开启以后才能启动下拉刷新

    回调函数
    @loadmore：下滑到底时的回调，回调中的参数是下一页的页码
    @refresh：下拉刷新
```
***
github链接
[链接名称](https://github.com/tanagag/vue-loadmore-simple)
