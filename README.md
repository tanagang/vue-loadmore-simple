基于vue开箱即用的全局组件下拉加载更多
### 使用方法
```js
    首先安装：npm install vue-loadmore-simple -S
    //main.js
    import {LoadMore} from 'vue-loadmore-simple'
    Vue.use(LoadMore)
    //template
    <template>
        <load-more :pageIndex="pageIndex" :pageSize="pageSize" :totalCount="totalCount" :callback="loadmore">
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
            }
        },
    }
    </script>
```
```
    参数：
    pageIndex：页码
    pageSize：页的大小
    totalCount：总条数
    tips：当没有数据时提供的文本（默认值：暂无数据）
    tipsSrc：当没有数据时提供的图标，必须是在线地址图标
    callback：下拉到底时的回调，参数是下一页的页码
```
***
github链接
[链接名称](https://github.com/tanagag/vue-loadmore-simple)
