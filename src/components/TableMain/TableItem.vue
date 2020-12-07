<template>
    <div>
        <template v-if="$scopedSlots[item.prop]">
            <div :class="{'content-data':!item.mobile}" >
                <slot :name="item.prop" :row="data"></slot>
            </div>
        </template>
        <!-- 没有插槽时 -->
        <template v-else>
            <span v-html="noSlotData"></span>
        </template>
    </div>
</template>

<script>
export default {
    props:{
        item:{
            type:Object,
            default:()=>{return {}}
        },
        data:{
            type:Object,
            default:()=>{return {}}
        },
        noSlotData:null
    },
}
</script>

<style scoped lang="scss">
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
</style>
