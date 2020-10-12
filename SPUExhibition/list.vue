<template>
    <faci-widget-frame :debounce="100" class="sample-product-list" mode="wide" :style="{maxHeight:cHeight}">
        <div class="sample-product-list__wrapper">
            <div class="sample-product-list">
                <div v-for="(item,i) in listData" :key="item._id" class="sample-product-grid__item">
                    <div :style="{backgroundImage: `url('${getFscLink(item.picture)[0]}')`}" class="sample-product-grid__img" @click="showDetail(i)">
                        <div class="sample-product__propertytitle">
                            <h1 class="sample-product__propertytitle-item sample-product__propertytitle-title">{{item.name || '--'}}</h1>
                            <p class="sample-product__propertytitle-item">{{item.standard_price || '--'}}</p>
                        </div>
                    </div>
                </div>
            </div>
            <fx-pagination
                @current-change="handleCurrentChange"
                :current-page.sync="currentPage"
                :page-size="searchQueryInfo.limit"
                class="sample-product-list__pagination"
                layout="prev, pager, next, jumper"
                :total="totalNum">
            </fx-pagination>
        </div>
    </faci-widget-frame>
</template>
<script>

export default {
    props:['maxPrice','searchKey','isReset','isRefresh'],
    data() {
        return {
            listData: [],
            searchQueryInfo: {
                limit: 20,
                offset: 0,
                filters: [],
                orders: [{
                    fieldName: "last_modified_time",
                    isAsc: false
                }]
            },
            currentPage: 1,
            totalNum: 0
        }
    },

    created() {
        this.parseListdata()
    },
    computed : {
        cHeight() {
            return $('.paaslayout-content-area').height() + 'px'
        }
    },
    watch: {
        maxPrice(cur, old) {
            if(cur === 0 || !cur) {
                this.searchQueryInfo.filters[0] = null
            }else{
                this.searchQueryInfo.filters[0] = {field_name:"standard_price",field_values:[cur],operator:"LTE"}
            }
            this.parseListdata();
        },
        searchKey(cur, old) {
            if(cur === 0 || !cur) {
                this.searchQueryInfo.filters[1] = null
            }else{
                this.searchQueryInfo.filters[1] = {field_name:"name",field_values:[cur],operator:"LIKE"}
            }
            this.parseListdata();
        },
        isReset(cur, old) {
            this.searchQueryInfo = Object.assign(this.searchQueryInfo, {
                limit: 20,
                offset: 0,
                filters: [],
                orders: [{
                    fieldName: "last_modified_time",
                    isAsc: false
                }]
            })
            this.parseListdata();
        },
        isRefresh(cur, old) {
            this.parseListdata();
        }
    },
    
    methods:{
        parseListdata () {
            let me = this;
            this.searchQueryInfo.offset = this.searchQueryInfo.limit * this.currentPage - this.searchQueryInfo.limit;
            let params = [{
                name: 'apiname',
                type: 'String',
                value: 'SPUObj'
            },{
                name: 'search_query_info',
                type: 'Map',
                value: this.searchQueryInfo
            }];
            FaciUI.userDefine.call_controller('Get_List__c', {
                params
            }).then(res => {
                if(res.Value.functionResult && res.Value.functionResult.success){
                    me.listData = res.Value.functionResult.data && res.Value.functionResult.data.dataList;
                    me.totalNum = res.Value.functionResult.data && res.Value.functionResult.data.total;
                }else if(res.Value.functionResult && res.Value.functionResult.errorMessage) {
                    this.$message({
                        message: res.Value.functionResult.errorMessage,
                        type: 'error'
                    })
                }
            })
        },

        showDetail (index) {
            this.$emit('showDetail', this.listData[index])
        },

        getFscLink (data) {
            if(data && data.length > 0) {
                return data.map((item) => {
                    return CRM.util.getFscLink(`${item.path}.${item.ext}`)
                })
            }else{
                return ['https://a9.fspage.com/FSR/uipaas/empty-image.svg']
            }
        },

        handleCurrentChange (val) {
            this.currentPage = val;
            this.parseListdata()
        }
    }
}
</script>
<style lang="less">
    .sample-product-list {
        margin: 0 12px;
        .sample-product-list__wrapper{
            font-size: 14px;
            line-height: 14px;
            display: flex;
            padding:12px 12px;
            background: rgb(255, 255, 255);
            border: 1px solid rgb(221, 219, 218);
            border-radius: 4px;
            background-clip: padding-box;
            height: 100%;
            display: flex;
            flex-wrap: wrap;
            box-sizing: border-box;
            flex-direction: column;
            align-items: flex-start;
            box-shadow: 0 2px 2px 0 rgba(0, 0, 0, 0.10);
            .sample-product-list {
                height: 100%;
                overflow: auto;
                display: flex;
                flex-wrap: wrap;
                &::-webkit-scrollbar {
                    display: none;
                }
            }
            .sample-product-grid__item {
                flex: 1 1 auto;
                min-width: 220px;
                margin: 4px;
                cursor: pointer;
                .sample-product-grid__img {
                    position: relative;
                    height: 220px;
                    background-size: cover;
                    background-position: 50%;
                    background-repeat: no-repeat;
                    .sample-product__propertytitle {
                        position: absolute;
                        bottom: 0;
                        left: 0;
                        right: 0;
                        color: #fff;
                        background-color: rgba(0,0,0,.3);
                        padding: 6px 8px;
                        .sample-product__propertytitle-item {
                            line-height: 19px;
                        }
                        .sample-product__propertytitle-title{
                            font-size: 16px;
                            line-height: 24px;
                            white-space: nowrap;
                            overflow: hidden;
                            text-overflow: ellipsis;
                        }
                    }
                }
            }
            .sample-product-list__pagination{
                width: 100%;
            }
        }
    }
</style>