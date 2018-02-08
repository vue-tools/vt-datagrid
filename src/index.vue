<style scoped>
    .vt-dg__warp{
        border: 1px solid #DEDEDE;
        border-top:0;
        min-height: 200px;
        position: relative;
        padding-bottom: 32px;
        box-sizing: border-box;
    }
    .vt-dg__table{
        height: 100%;
    }
    .vt-dg__pager{
        width: 100%;
        display: block;
        position: absolute;
        bottom: 0;
        left: 0;
    }
</style>
<style>
    .vt-dg__pager .ui-pagination--total{
        margin: 0 5px;
        line-height: 28px;
    }
    .vt-dg__pager select.ui-counter{
        height: 28px;
        margin: 0 5px;
        border: 1px solid #dedede;
        padding:0 5px;
        box-sizing: border-box;
    }
    .vt-dg__pager .ui-button{
        margin: 0 5px;
    }
    .vt-dg__pager .ui-pager{
        position: relative;
        top: 1px;
    }
</style>
<template>
    <div  class="vt-dg__warp">
        <Tables class="vt-dg__table" :data="vo.rows" @selection-change="selectionChange" :selection="selection">
            <slot></slot>
        </Tables>

        <Pagination class="vt-dg__pager" :total="vo.pagination.total" :count="po.pageSize" :currentPage="po.pageNumber"
                    :countOptions="vo.pagination.countOptions" @current-change="pageChange"
                    @count-change="pageSizeChange">
        </Pagination>
    </div>
</template>

<script>
    let noop = ()=> {
    }
    let options = {
        post: noop,
        get: noop,
        pageNumber: 1,
        pageSize: 20,
        pageList: [10, 20, 50, 100, 200],
        requestInterceptor: noop,
        responeseInterceptor: function (data) {
            return data
        }
    }
    let $set = (values)=> {
        for (let k in values) {
            options[k] = values[k]
        }
    }
    //    let dgList = []


    export default {
        data(){
            return {
                po: {
                    pageNumber: options.pageNumber,
                    pageSize: options.pageSize
                },
                vo: {
                    height:0,
                    rows: [],
                    pagination: {
                        total: 0,
                        countOptions: options.pageList
                    },
                    loading: false
                }
            }
        },
        props: {
            params: {
                type: Object,
                default: function () {
                    return {}
                }
            },
            url: String,
            method: {
                type: String,
                default: 'get'
            },
            pageNumber: Number,
            pageSize: Number,
            selection: Boolean,
            pageList: {
                type: Array,
                default: function () {
                    return options.pageList
                }
            },
            requestInterceptor: Function,
            responeseInterceptor: Function
        },
        methods: {
            selectionChange(data){
                this.$emit("selectionChange", data)
            },

            pageChange(page){
                if (page !== this.po.pageNumber) {
                    this.po.pageNumber = page
                    this.loadData()
                }
            },
            pageSizeChange(size){
                this.po.pageSize = size
                this.loadData()
            },
            loadData(){
                let request = options[this.method]
                let params = Object.assign({}, JSON.parse(JSON.stringify(this.params)), this.po)
                let requestInterceptor = this.requestInterceptor || options.requestInterceptor
                if (requestInterceptor && requestInterceptor != noop) {
                    params = requestInterceptor(params)
                }
                this.vo.loading = true
                request(this.url, params).then(res=> {
                    this.vo.loading = false
                    let responeseInterceptor = this.responeseInterceptor || options.responeseInterceptor
                    if (responeseInterceptor && typeof responeseInterceptor === "function") {
                        let data = responeseInterceptor(res)
                        this.vo.rows = data.rows
                        this.vo.pagination.total = data.total
                    } else {
                        this.vo.rows = res
                    }
                }).catch(err=> {
                    this.vo.rows=[]
                    this.vo.loading = false
                })
            },
        },
        created() { //把组件所在页面的数据复制到当前页面
            for(let prop in this.$parent) {
                if(!this.hasOwnProperty(prop)) {
                    console.info(prop)
                    this[prop] = this.$parent[prop]
                }
            }
        },
        mounted: function () {
//            dgList.push(this)
        },
        components: {

        },
        watch: {
            'pageNumber': function (val) {
                this.po.pageNumber = val
                console.info("cccc")
            },
            'pageSize': function (val) {
                this.po.pageSize = val
            }
        },
        filters: {},
        $set
    }
</script>