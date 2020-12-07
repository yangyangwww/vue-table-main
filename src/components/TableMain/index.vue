<template>
    <div class="table-main"  v-loading='loading' :class="{'table-border':!border}">
        <div class="table-wrapper"  ref='tableWrapper'  v-if="device!='mobile'" :class="{'table-wrapper-fix':scrollHeight}">
        <!-- 表头 -->
        <div class="table-header" :style="{'top':scrollHeight?(clientWidth+'px'):'auto'}" :class="{'table-width':tableHeight}" >
            <table cellpadding="0"  cellspacing="0" class="table table-header-main" style="background:#fff;position: relative;"  :style="{'left':'-'+scrollLeft+'px'}">
                <tr>
                    <th class="tab-header checkout-pading" width="30" v-if="selection">
                        <checkbox  :disabled='dataList.every(a=>a.disabled)' :class="{'is-indeterminate':(checkList.length != data.length && checkList.length>0)}"
                            v-model="allCheck" :label='true'></checkbox>
                    </th>
                    <template v-for="(item,index) in schemaList">
                        <th class="tab-header"  @click.stop="getSort(item)" :style="'max-width:'+item.width+'px; min-width:'+item.width+'px;'" :key='index'
                            :width="item.width">
                            <div class="header-content">
                                {{item.label}} 
                                <div class="header-sort" v-if="item.sortName" @click.stop="getSort(item)">
                                    <i class="el-icon-caret-top" @click.stop="sortDown(item)" :class="{'color-blue':currentSort && currentSort[item.sortName] == 'down'}" ></i>
                                    <i class="el-icon-caret-bottom" @click.stop="sortUp(item)" :class="{'color-blue':currentSort && currentSort[item.sortName] == 'up'}" ></i>
                                </div>
                            </div>
                        </th>
                    </template>
            
                </tr>
            </table>
        </div>
          
            <div class="noData" :class="{'noDataHeight':!$scopedSlots.noData}" v-if="!loading && !list.length">
                <template v-if="$scopedSlots.noData">
                    <slot name="noData"></slot>
                </template>
                <template v-else>
                    <i class="iconfont icon-kong"></i> <br>
                    暂无数据
                </template>
            </div>
            <div class="table-body" v-if="list.length" ref='tableBody' :style="'max-height:'+(currentHigth || height)+'px;'"
                @mousemove="onMousemove" @mousedown="onMousedown" @mouseup="onMouseup">
                <table v-for="(listItem,index) in list" :key='index' cellpadding="0" cellspacing="0"
                   class="table"  :class="{'tab_top':spacing,'tab-border':!spacing}">
                    <!-- 有 header 的插槽才会显示这列 -->
                    <tr v-if="$scopedSlots.header">
                        <!-- 二维数组的时候才会显示 header 这一列才会显示多选框-->
                        <td width="30" :class="{'is-indeterminate':isIndeterminate(listItem)}" v-if="selection && groupName"
                            class="header-check text-align checkout-pading">
                            <checkbox :disabled='getChildren(listItem).every(a=>a.disabled)' v-model="groupCheck"
                                :label="listItem.group" @change='groupChange(listItem)'></checkbox>
                        </td>
                        <td class="schema-header" height="40" :colspan="groupName && selection?schemaList.length:schemaList.length+1">
                            <slot name="header" :row="listItem"></slot>
                        </td>
                    </tr>
                    
                    <tr class="tab-row" :class="[currentId === (datas.id || datas._id)?'activeTab':'',rowClass(datas)]" v-for="(datas,key) in getChildren(listItem)"
                        :key='key' @click="onRowClick(datas,key)">
                        <td width="30" @click.stop v-if="selection" class="text-align checkout-pading">
                            <checkbox v-model="checkList" :label="datas.id || datas._id" :disabled='datas.disabled'
                                @change='childrenChange(listItem,datas)'></checkbox>
                        </td>
                        <template v-for="(item,k) in schemaList">
                            <td class="tab-text" :style="'max-width:'+item.width+'px; min-width:'+item.width+'px;'"
                                :align="item.align" :key='k' :width="item.width">
                                <!--schema里的 tooltip 为true才会显示 -->
                                <el-tooltip class="item" :open-delay="500" effect="dark" :disabled='!item.tooltip' placement="right" popper-class="table-main-tooltip">
                                    <div :class="{'white-space':item.tooltip}">
                                        <TableItem :data='datas' :item='item' :noSlotData="$options.filters['emptyFilter'](listItem[item.prop])">
                                            <slot :slot="item.prop" :name="item.prop" :row="datas"></slot>
                                        </TableItem>
                                    </div>
                                    <div slot='content' style="max-width: 500px;line-height: 20px;" v-if="item.tooltip">
                                        <TableItem :data='datas' :item='item' :noSlotData="$options.filters['emptyFilter'](listItem[item.prop])">
                                            <slot :slot="item.prop" :name="item.prop" :row="data"></slot>
                                        </TableItem>
                                    </div>
                                </el-tooltip>
                            </td>
                        </template>
                    </tr>
                </table>
            </div>
        </div>



        <!-- 移动端时显示的内容 -->
        <div class="mobile-wrapper" v-else>
            <div v-if="selection" class="mobile-all">
                <checkbox :disabled='dataList.every(a=>a.disabled)' :class="{'is-indeterminate':(checkList.length != data.length && checkList.length>0)}"
                    v-model="allCheck" :label='true'>全选</checkbox>
            </div>
            <template v-for="(data,index) in dataList">
                <div class="mobile-content" :key='index'  @click.stop="onRowClick(data,index)">
                    <el-checkbox class="checkbox-content" @click.stop v-model="checkList" v-if="selection" :label="data.id"
                        :disabled='data.disabled' @change='mobileChange(data)'>{{null}}</el-checkbox>
                    <template v-for="(item,key) in schemaList">
                        <div :key='key' v-if="item.prop != 'action'" :style="selection?'margin-left:12px':''" class="mobile-main">
                                <div class="display-content label" v-if="!item.mobile">{{item.label}}：</div>
                                <div class="display-content ">
                                    <TableItem :data='data' :item='item' :noSlotData="$options.filters['emptyFilter'](data[item.prop])">
                                        <slot :slot="item.prop" :name="item.prop" :row="data"></slot>
                                    </TableItem>
                                </div>
                        </div>
                        <!-- 操作显示的内容 -->
                        <div :key='key' v-if="item.prop == 'action'" class="actions">
                            <slot :name="item.prop" :row="data"></slot>
                        </div>
                    </template>
                </div>
            </template>

            <div class="mobile-content noData" v-if="!list.length">
                <i class="iconfont icon-kong"></i> </br>
                暂无数据
            </div>
        </div>
        
        <div class="flooter" v-if='showPage'>
            <div class="floot-control">
                <slot name="control"></slot>
            </div>
            <div class="floot-page">
                <pagination :total="total" :layout="layout" :small="paginationSmall" :pageIndex.sync="listQuery.pageIndex"
                    :limit.sync="listQuery.pageSize" @pagination="getList" />
            </div>
        </div>


    </div>
</template>

<script>
import Pagination from '@/components/Pagination';
import TableItem from './TableItem';
export default {
 name:'TableMain',
 components:{Pagination,TableItem},
    props:{
        // 表头具体内容，以及插槽的name字段
        schema: {
            type: Array,
            default: () => {
                return []
            }
        },
        api: null, //列表的api
        data: {
            type: Array,
            default: () => {
                return []
            }
        },
        // selection 为true的时候 就显示多选框
        selection: {
            type: Boolean,
            default: false
        },
        listQuery: {
            type: Object,
            default: () => {
                return {
                    pageIndex: 1,
                }
            }
        },
        paginationSmall: {
            type: Boolean,
            default: false
        },
        // 初始接口渲染字段层级  {totalKey：result.total,listKey:result.data}
        config: {
            type: Object,
            default: () => {
                return {}
            }
        },
        removePager: {
            type: Boolean,
            default: false
        },
        height: {
            type: String,
            default: ''
        },
        //  groupName 有数据就组装成二维数组
        groupName: {
            type: String,
            default: ''
        },
        showPage: {
            type: Boolean,
            default: true
        },
        // true 为显示表格之间的间距， false 没有间距
        spacing: {
            type: Boolean,
            default: true
        },
        // 没有api时 需要传total
        currnetTotal: {
            type: Number,
            default: 0
        },
        // border 为false是td和tr就没有边框
        border:{
            type: Boolean,
            default: false
        },
        // 在表格的每一行添加class
        rowClass:{
            type:Function,
            default:()=>{}
        },
        // 是否为弹窗组件，如果是弹窗组件就不需要固定表头
        dailog:{
            type: Boolean,
            default: false
        }
    },
    watch: {
        data(val) {
            if (!this.api) {
                this.dataList = this.data
                this.total = this.currnetTotal
                this.list = this.getData();
            } else {
                this.getList()
            }
        }
    },
    data() {
        return {
            list: [],
            checkList: [], //选中的分组下children的数据
            groupCheck: [], //选中的分组
            // allCheck:false,
            loading: false,
            layout: '',
            currentId: '',
            total: 0,
            tableHeight: false,
            mouseEvent: {
                sx: 0,
                sy: 0,
                ex: 0,
                ey: 0,
                moveable: false,
                isMoved: false //是否移动了，如果移动了，就阻止click事件
            },
            currentHigth: '',
            dataList: [],
            scrollHeight:false,
            clientWidth:null,
            scrollLeft:0,
            currentSort:{},
            sortGroupList:[],
            device:""
        }
    },
    created() {
        this.init()
        this.currentId = ''
    },
    computed: {
        allCheck: {
            get() {
                // this.data.length
                return !!this.checkList.length && this.checkList.length == this.dataList.length
            },
            set(val) {
                if (val) {
                    this.checkList = this.dataList.filter(a => !a.disabled).map(a => a.id || a._id)
                    this.groupCheck = this.list.filter(a => a.children && !a.children.every(b => b.disabled)).map(
                        a => a.group)
                } else {
                    this.checkList = []
                    this.groupCheck = []
                }
                this.$emit('rowCheck', this.checkList && this.checkList.length ? this.dataList.filter(a => !a.disabled) : [])
            }
        },
        params: {
            get() {
                return this.listQuery
            },
        },
        schemaList:{
            get() {
                return this.schema
            },
        },
    },
    beforeMount() {
        document.addEventListener('scroll', this.resizeScrollHeight, true);
        document.addEventListener('scroll',this.tableScroll,true);
        window.addEventListener('resize', this.getMobile);
    },
    beforeDestroy () {
        document.removeEventListener("scroll", this.resizeScrollHeight);
        document.removeEventListener('scroll',this.tableScroll);
        window.removeEventListener('resize', this.getMobile)
    },
    methods: {
        init() {
            if (!this.api) {
                this.dataList = this.data
                this.total = this.currnetTotal
                this.list = this.getData();
            } else {
                this.getList()
            }
            if (this.removePager) {
                this.layout = 'total, sizes, prev, next'
            } else {
                this.layout = 'total, prev, pager, next'
                if (this.device != 'mobile') {
                    this.layout += ', sizes, jumper'
                }
            }
            
        },
        getList() {
            this.loading = true;
            this.api(this.params).then(res => {
                const listKey = this.config && this.config.listKey ? this.config.listKey : 'result.data',
                    totalKey = this.config && this.config.totalKey ? this.config.totalKey : 'result.total';
                this.dataList = eval('res.' + listKey)
                this.list = this.getData();
                this.total = eval('res.' + totalKey);
                this.loading = false;
                this.$emit('success', this.dataList);

            }).catch((err) => {
                this.loading = false;
            })

        },
        getMobile(){
            if(document.body.clientWidth <= 600){
                this.device = 'mobile'
            }else{
                this.device = ''
            }
        },
        scrollBar() {
            // 处理表格的滚动条
            setTimeout(() => {
                const height = this.currentHigth || this.height
                // 适配表格滚动条
                if(!height){
                    return
                }
                if (this.$refs.tableBody && this.$refs.tableBody.scrollHeight > Number(height)) {
                    this.tableHeight = true
                }else{
                    this.tableHeight = false
                }
            })
        },

        //  组装二维数组
        getData() {
            this.scrollBar()
            // 数据里没有id的时候 就手动加个id
            for (const i in this.dataList) {
                const item = this.dataList[i]
                if (!item.id) {
                    item['_id'] = this.genNonDuplicateID()
                }
            }
            if (!this.groupName) {
                return this.dataList
            }
            const list = []
            this.dataList.forEach(item => {
                const sameGroup = []
                const current = list.find(items => items.group == item[this.groupName])
                if (!current) {
                    list.push({
                        group: item[this.groupName],
                        children: [item]
                    })
                } else {
                    current.children.push(item)
                }

            })
            this.sortGroupList = list.sort((a, b) => a.group - b.group);
            return list.sort((a, b) => a.group - b.group);
        },
        onRowClick(row, index) {
            this.currentId = row.id || row._id
            if (!this.mouseEvent.isMoved) {
                this.$emit('row-click', row)
            }
        },
        // 点击table的一行即可变为选中状态
        toggleRowSelection(row, index) {
            if (!this.checkList.some(item => item == row.id)) {
                this.checkList.push(row.id);
            } else {
                this.checkList = this.checkList.filter(item => item != row.id);
            }
            return this.dataList.filter(item => {
                return this.checkList.indexOf(item.id || item._id) > -1
            })

        },
        mobileChange(row) {
            // 移动端的每一行的点击事件
            this.$emit('rowCheck', this.dataList.filter(item => {
                return this.checkList.indexOf(item.id || item._id) > -1
            }))
        },
        // 分组全选
        groupChange(row) {
            let opt = {}
            // 拿到当前点击的data
            opt = this.list.find(item => item.group == row.group)
            // 判断当前的data里有没有children数组
            if (opt && opt.children && opt.children.length) {
                opt.children.forEach(item => {
                    if (!item.disabled) {
                        this.checkList.push(item.id)
                    }
                    // 当前点击的分组下的children数组全选操作
                })

                if ((!this.groupCheck && !this.groupCheck.length) || this.groupCheck.map(item => item).indexOf(row.group) == -1) {
                    const list = []
                    this.checkList.forEach(item => {
                        // 取消全选操作
                        if (!opt.children.some(items => items.id == item)) {
                            list.push(item)
                        }
                    })
                    this.checkList = list
                }
            }

            this.$emit('rowCheck', this.dataList.filter(item => {
                return this.checkList.indexOf(item.id || item._id) > -1
            }))

        },
        // 单个选择
        childrenChange(row, childrenRow) {
            if (this.groupName && this.selection) {
                const opt = this.list.find(item => item.group == row.group)

                // 有一个 没选中，上级的选中的状态为false
                if (!this.checkList.some(items => items == childrenRow.id || childrenRow._id) && this.groupCheck.map(item => item).indexOf(row.group) > -1) {
                    this.groupCheck.splice(this.groupCheck.map(item => item).indexOf(row.group), 1)
                }
                const list = []
                // 同一个children里的数据都选中，父级就跟着选中
                this.checkList.forEach(item => {
                    if (row.children.some(items => items.id || items._id == item)) {
                        list.push(item)
                    }
                })

                if (list.length == row.children.length) {
                    this.groupCheck.push(row.group)
                }
            }

            this.$emit('rowCheck', this.dataList.filter(item => {
                return this.checkList.indexOf(item.id || item._id) > -1
            }))
        },
        // 鼠标点击弹起之后触发的事件
        onMousedown(e) {
            // isMoved 控制是否 组织click事件
            this.mouseEvent.moveable = true;
            this.mouseEvent.isMoved = false;
            this.mouseEvent.sx = this.mouseEvent.ex = e.pageX;
            this.mouseEvent.sy = this.mouseEvent.ey = e.pageY;
        },
        // 鼠标移动时的事件
        onMousemove(e) {
            if (this.mouseEvent.moveable) {
                this.mouseEvent.ex = e.pageX; //x坐标
                this.mouseEvent.ey = e.pageY; //y坐标
            }
        },
        // 鼠标松开时触发的事件
        onMouseup() {
            // ex 是鼠标落下去的位置   sx是鼠标弹起时的位置
            this.mouseEvent.moveable = false;
            //横坐标移动超过10px，
            if (Math.abs(this.mouseEvent.ex - this.mouseEvent.sx) >= 5) {
                this.mouseEvent.isMoved = true;
            }
        },
        getChildren(data) {
            return data.children || [data]
        },
        // 控制父级数组的checkbox的显示状态
        isIndeterminate(row) {
            const item = this.getChildren(row).filter(dataItem => {
                return this.checkList.indexOf(dataItem.id || dataItem._id) > -1
            })
            if (item.length != this.getChildren(row).length && item.length > 0) {
                return true
            }
            return false
        },
        sortDown(row){
            // 顺序
            if(!this.dataList.length){return}
            if(this.currentSort[row.sortName] == 'down'){
                // 再次点击时还原排序之前的顺序
                this.reductionSort()
                return
            }
            
            this.currentSort = {}
            this.currentSort[row.sortName] = 'down'
            let sortList = []
              if(this.groupName){
                  // 二维数组排序
                 sortList = this.deepClone(this.sortGroupList)
                 sortList.forEach(item=>{
                    item.children = this.sortDownOrder(item.children,row)
                 })
              }else{
                  sortList = this.deepClone(this.dataList)
                  this.list = this.sortDownOrder(sortList,row)
              }
        },
        sortDownOrder(sortList,row){
            sortList.sort((a,b)=>{
                if(typeof a[row.sortName] == 'number'){
                    return a[row.sortName] - b[row.sortName]
                }else{
                    return a[row.sortName].localeCompare(b[row.sortName])
                }
            })
            return sortList
        },
        sortUpOrder(sortList,row){
            sortList.sort((a,b)=>{
                if(typeof a[row.sortName] == 'number'){
                    return b[row.sortName] - a[row.sortName]
                }else{
                    return b[row.sortName].localeCompare(a[row.sortName]);
                }
            })
            return sortList
        },
        sortUp(row){
            // 逆序
            if(!this.dataList.length){return}
            if(this.currentSort[row.sortName] == 'up'){
                // 再次点击时还原排序之前的顺序
                this.reductionSort()
                return
            }
            this.currentSort = {}
            this.currentSort[row.sortName] = 'up'
            
            let sortList = []
              if(this.groupName){
                  // 二维数组排序
                 sortList = this.deepClone(this.sortGroupList)
                 sortList.forEach(item=>{
                    item.children = this.sortUpOrder(item.children,row)
                 })
              }else{
                  sortList = this.deepClone(this.dataList)
                  this.list = this.sortUpOrder(sortList,row)
              }
        },
        reductionSort(){
            // 还原
            this.currentSort = {}
            this.list = this.groupName?this.sortGroupList:this.dataList
        },
        getSort(row){
            // 排序
            if(!this.dataList.length || !row.sortName){return}
            // 在schema里设置 sortName 就可以排序
            const sortList = this.deepClone(this.dataList)
            // hasOwnProperty() 方法会返回一个布尔值，指示对象自身属性中是否具有指定的属性（也就是，是否有指定的键）。
            if(Object.keys(this.currentSort).length==0 || !this.currentSort.hasOwnProperty(row.sortName)){
                // 顺着排序
                this.sortDown(row)
            }else if(this.currentSort[row.sortName] == 'down'){
                // 逆着排序
                this.sortUp(row)
            }else{
                // 还原
                this.reductionSort()
            }
            
        },
        
        tableScroll(){
            // 表头固定的时候出现横向滚动条时 表头也跟着滚动
            if(this.scrollHeight){
                this.scrollLeft = this.$refs.tableWrapper && this.$refs.tableWrapper.scrollLeft || 0
            }
        },
        getPosition(dom) {
            var t = dom.offsetTop,
                l = dom.offsetLeft;
            let obj = dom;
            //从目标元素开始向外遍历，累加top和left值  
            for (var t = obj.offsetTop, l = obj.offsetLeft; obj = obj.offsetParent;) {
                t += obj.offsetTop;
                l += obj.offsetLeft;
            }
            return {
                left: l,
                top: t
            }
        },
        resizeScrollHeight(){
            if(document.documentElement.scrollTop == 0){
                 this.scrollHeight = false
                 return
            }
            // 监听滚动条，滚动条离顶部的距离大于头部的高度时让表头固定定位
            this.clientWidth = document.documentElement.clientWidth > 1200?100:60
            // console.log(this.clientWidth)
            const scrollTop = document.documentElement.scrollTop + this.clientWidth 
            const offsetTop = this.$refs.tableWrapper && this.getPosition(this.$refs.tableWrapper).top || 0
            
            if(scrollTop < offsetTop){
                 this.scrollLeft = 0
            }
            
            if(scrollTop > offsetTop){
                this.scrollHeight = true
            }else{
                this.scrollHeight = false
            }
            
        },
        resizeHigth(num) {
            this.currentHigth = num
        },
        // 搜索时重置pageIndex
        query() {
            this.params.pageIndex = 1
            this.getList()
        },
        genNonDuplicateID(randomLength) {
            return Number(Math.random().toString().substr(3, 6) + Date.now()).toString(36).substr(0,randomLength)
        },
        deepClone(source) {
            if (!source || typeof source !== 'object') {
                return source;
            }
            const targetObj = source.constructor === Array ? [] : {}
            Object.keys(source).forEach(keys => {
                if (source[keys] && typeof source[keys] === 'object') {
                    targetObj[keys] = this.deepClone(source[keys])
                } else {
                    targetObj[keys] = source[keys]
                }
            });
            return targetObj
        }
    }

}
</script>

<style scoped lang="scss">
    .table-width {
        padding-right: 6px;
    }

    .table-body {
        overflow-y: auto;
        overflow-x: hidden;
        display: inline-block;
        min-width: 100%;
        vertical-align: top;
    }

    .table-wrapper {
        overflow-x: auto;
        overflow-y: hidden;
        width: 100%;
    }
    .table-wrapper-fix{
        padding-top: 42px;
        .table-header{
            position: fixed;
            width: 1160px;
            z-index:100;
            overflow: hidden;
        }
        
    }
  
   
    .tab_top {
        margin-top: 10px;
    }

    .tab-border {
        border-top: snow;
        margin-top: -1px;
    }
    .table-border{
        .table-body{padding-bottom: 1px;}
        .table{
            border: 1px solid #e7e7e7;
             // border-top:none;
             // margin-bottom: -1px;
            td,
            th {
                border: none;
            } 
        }
     
    }
    .table.table-header-main{
        border: 1px solid #e7e7e7;
        td,
        th {
            border: none;
        } 
    }
    .table {
        border-collapse: collapse;
        width: 100%;
        
        td,
        th {
            border-collapse: collapse;
            border: 1px solid #e7e7e7;
            // border-spacing: 0;
        }

        .tab-header {
            text-align: center;
            line-height: 40px;
            font-size: 12px;
            background: #FAFAFA;
            color: #434343;
            // .header-content{
            //     overflow: hidden;
            //     text-overflow: ellipsis;
            // }

            // border: 0;
            .header-sort{
                display: inline-flex;
                flex-direction: column;
                vertical-align: middle;
                cursor: pointer;
                color:#aaa;
                // border: 1px solid blue;
                i{
                    // border: 1px solid red;
                    font-size: 10px;
                    vertical-align: bottom;
                    line-height: 0.6;
                }
            }
        }

        .schema-header {
            background: #f5f5f5;
            padding: 0 10px;
        }

        .tab-text {
            padding: 10px;
            box-sizing: border-box;
            overflow: hidden;
        }

        .header-check {
            background: #f5f5f5;
        }

        .text-align {
            text-align: center;
        }
    }

    .flooter {
        div {
            display: inline-block;
        }

        .floot-control {
            margin-top: 20px;
        }

        .floot-page {
            float: right;
        }
    }

    .tab-row {
        background-color: white;

        &:hover {
            // background-color:#fdfcfc ;
            background-color: #F5F7FA;
        }
    }

    .activeTab {
        background-color: #fffbf5;

        &:hover {
            background-color: #fffbf5;
        }
    }

    .is-indeterminate {
        /deep/ {
            .el-checkbox__inner {
                background-color: #ff8d1a;
                border-color: #ff8d1a;

                &:before {
                    content: "";
                    position: absolute;
                    display: block;
                    background-color: #fff;
                    height: 2px;
                    transform: scale(.5);
                    left: 0;
                    right: 0;
                    top: 4px;
                }

                &:after {
                    display: none;
                }
            }
        }
    }

    .table-main {
        /deep/.el-checkbox__label {
            padding: 0;
        }

        .noData {
            text-align: center;
            color: #cccccc;
            border: 1px solid #e7e7e7;
            border-top: none;
        }

        .noDataHeight {
            // height: 100px;
            padding: 25px;
        }
    }

    .checkout-pading {
        padding: 0 5px;
    }







    .mobile-wrapper {
        .mobile-all {
            margin-bottom: 10px;

            /deep/.el-checkbox__label {
                padding-left: 5px;
            }
        }

        .mobile-content {
            width: 100%;
            border: 1px solid #e7e7e7;
            box-shadow: 0px 0px 1px 0px rgba(0, 0, 0, 0.1);
            padding: 15px 15px 10px 15px;
            margin-bottom: 20px;
            position: relative;

            .mobile-main {
                // white-space: nowrap;
                overflow: hidden;
                position: relative;
                overflow: hidden;
            }

            .checkbox-content {
                position: absolute;
                top: 50%;
                left: 5px;
                transform: translateY(-50%);
                // z-index: 100;
            }

            .content-right {
                margin-right: 55px
            }

            .display-content {
                display: initial;
                vertical-align: middle;
                line-height: 20px;
                vertical-align: top;

                &.label {
                    color: #999;
                    min-width: 70px;
                    display: inline-block;
                    vertical-align: top;
                    margin-right: 5px;
                }
            }

            .content-data {
                height: 25px;
                overflow: hidden;
                display: inline;
                vertical-align: middle;

                div {
                    display: inline;
                }

                /deep/.el-input-number {
                    width: 80px!important;
                }
            }

            .btn-detail {
                position: absolute;
                right: 0;
                top: -6px;
            }

            .actions {
                border-top: 1px solid #eee;
                margin-top: 10px;
                text-align: right;
                padding-top: 5px;

                div {
                    display: inline-block;
                    margin-left: 10px;

                    /deep/.el-button {
                        font-size: 14px;
                    }
                }
            }
        }

    }
    
    .showTip{
        div{
            white-space:normal!important;
        }
    }
    
    
    // @media (max-width: 1200px){
    //     .table-wrapper-fix{
    //         .table-header{
    //             width: 100%!important;
    //             // margin: 0 20px;
    //         }
            
    //     }
    // }
</style>
<style lang="scss">
.table-main-tooltip{
    .tip-hidden{
        display: none;
    }
}
.el-dialog__wrapper,.el-drawer{
    .table-wrapper-fix{
        padding-top: 0;
        .table-header{
            position: static;
            width:auto;
        }
        
    }
}

</style>
