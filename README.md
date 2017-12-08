# vt-datagrid
vue表格分页组件  
 
# Use 
```shell
$ npm install vt-datagrid --save
```

```shell
 import Datagrid from 'vt-datagrid'
 
 Vue.component('Datagrid', Datagrid)
```

# Dependencies
 
该组件依赖 [vt-table](https://github.com/vue-tools/vt-table)组件和
[vt-pagination](https://github.com/vue-tools/vt-pagination)组件 
 

# Api 
 
### Props 
*   `params` 查询参数
*   `url` 请求的地址
*   `method`请求的方式 `get post`,默认 `get`
*   `pageNumber` bool类型，控制弹窗是否可以拖动，默认 `true` 
*   `pageSize` bool类型，控制弹窗是否可以拖动，默认 `true` 
*   `selection` 是否有复选框 `false` 
*   `pageList`  当前页面可以展示多少条数据的数组



### Event 
*   `selectionChange` 当用复选框时，选择一行时触发的事件

### Setting

 使用$set静态方法来设置项目中的全局设置。例如：
 ```
 
 ```