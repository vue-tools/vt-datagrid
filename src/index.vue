<style scoped>
    .vt-dg__warp{
        background: red;
    }
</style>
<template>
    <div :style="{height:vo.height}" class="vt-dg__warp">
        <Tables :data="vo.rows" @selection-change="selectionChange" :selection="selection">
            <slot></slot>
        </Tables>
        <Pagination :total="vo.pagination.total" :count="po.pageSize" :currentPage="po.pageNumber"
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
    let dgList = []
    window.addEventListener("resize", function () {
        for (let dg of dgList) {
            dg.resize()
        }
    })

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
            resize(){
                let top = document.documentElement.scrollTop
                let h = this.$el.getBoundingClientRect().top
                this.vo.height = window.document.documentElement.clientHeight - h - 50
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
                        this.vo.data = res
                    }
                }).catch(err=> {
                    this.vo.loading = false
                })
            },
        },
        mounted: function () {
            dgList.push(this)
            this.$nextTick(()=> {
                this.resize()
            })
        },
        components: {

        },
        watch: {
            'pageNumber': function (val) {
                this.po.pageNumber = val
            },
            'pageSize': function (val) {
                this.po.pageSize = val
            }
        },
        filters: {},
        $set
    }
</script>