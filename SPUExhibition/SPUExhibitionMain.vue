<template>
    <faci-widget-frame :debounce="100" class="sample-product-wrapper" mode="wide">
        <div class="sample-product-wrapper__container">
            <filter-card 
                class="sample-product-filter"
                @handleMaxPriceChange="handleMaxPriceChange"
                @handleSearchKeyChange="handleSearchKeyChange"
                @reset="reset"/>
            <list 
                class="sample-product-list" 
                :maxPrice="maxPrice"
                :searchKey="searchKey"
                :isReset="flag"
                :isRefresh="refreshFlag"
                @showDetail="showDetail"/>
            <edit 
                class="sample-product-detail" 
                v-if="isShowDetail"
                :detailData="detailData"
                @refresh="refresh"
                @closeEditCard="closeEditCard"/>
            <fx-dialog
                ref="dialog1"
                :visible.sync="dialogVisible"
                width="80%"
                :append-to-body="true"
                :slider-panel="true"
                :modal="false"
                >
                    <fx-card style="height: 700px; overflow: scroll;">
                        <faci-detail apiName="SPUObj" :dataId="detailData._id"></faci-detail>
                    </fx-card>
            </fx-dialog>
        </div>
    </faci-widget-frame>
</template>
<script>
import list from './list.vue';
import edit from './edit'
import filterCard from './filterCard.vue';

export default {
    data() {
        return {
            isShowDetail:false,
            maxPrice:0,
            searchKey:'',
            flag: false, //reset刷新
            detailData: {},
            refreshFlag:false,
            dialogVisible: false
        }
    },
    methods:{
        showDetail(detailData) {
            this.isShowDetail = true;
            this.detailData = detailData;
        },

        closeEditCard() {
            this.isShowDetail = false;
        },

        handleMaxPriceChange(maxPrice) {
            this.maxPrice = maxPrice;
        },

        handleSearchKeyChange(searchKey) {
            this.searchKey = searchKey;
        },

        reset(maxPrice,searchKey) {
            this.maxPrice = maxPrice;
            this.searchKey = searchKey;
            // this.flag = !this.flag;
        },
        refresh() {
            this.refreshFlag = !this.refreshFlag;
        },
    },
    components: {
        list,
        filterCard,
        edit
    }
}
</script>
<style lang="less">
    .sample-product-wrapper{
        padding: 4px;
        .sample-product-wrapper__container {
            display: flex;
            .sample-product-filter {
                flex:1;
            }
            .sample-product-list {
                flex:5;
            }
            .sample-product-detail {
                flex:2;
            }
        }
    }
</style>