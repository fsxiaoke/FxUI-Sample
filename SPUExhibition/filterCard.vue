<template>
    <faci-widget-frame :debounce="100" class="sample-product-filter" mode="wide">
        <article class="filter-card">
            <header class="filter-card__header filter-grid">
                <div class="grid-icon">
                    <i class="el-icon-search"></i>
                    <div class="grid-icon-title">Filter</div>
                </div>
                <div class="grid-reset" @click="reset">
                    <fx-button size="mini">reset</fx-button>
                </div>
            </header>
            <div class="filter-card__body">
                <div class="filter-card__body-search-key">
                    <div class="filter-form-element__label">Search Key</div>
                    <fx-input 
                        v-model="inputValue" 
                        placeholder="请输入内容" 
                        class="filter-form-element" 
                        size="mini"
                        :debounce="200"
                        @change="handleSearchKeyChange"></fx-input>
                </div>
                <div class="filter-card__body-slider">
                    <div class="sld-max-price">
                        <div>Max Price - {{maxPrice}}</div>
                        <fx-slider
                            v-model="maxPrice"
                            :debounce="200"
                            :max="100"
                            @change="handleMaxPriceChange">
                        </fx-slider>
                    </div>
                    <div class="sld-max-count">
                        <div>自定义（无筛选效果） - {{maxCount}}</div>
                        <fx-slider
                            v-model="maxCount"
                            :debounce="200"
                            :max="100">
                        </fx-slider>
                    </div>
                </div>
            </div>
        </article>
    </faci-widget-frame>
</template>
<script>

export default {
    name:'filter-card',
    data() {
        return {
            maxPrice: 0,
            maxCount: 0,
            inputValue: ''
        }
    },
    methods:{
        handleMaxPriceChange() {
            this.$emit('handleMaxPriceChange',this.maxPrice)
        },

        handleSearchKeyChange () {
            this.$emit('handleSearchKeyChange',this.inputValue)
        },

        reset() {
            this.maxPrice = 0;
            this.maxCount =  0;
            this.inputValue = '';
            // this.$emit('handleMaxPriceChange',this.maxPrice)
            // this.$emit('handleSearchKeyChange',this.inputValue)
            this.$emit('reset',this.maxPrice,this.inputValue)
        }
    }
}
</script>
<style lang="less">
    .sample-product-filter{
        font-size: 14px;
        line-height: 14px;
        position: relative;
        padding: 0;
        background: rgb(255, 255, 255);
        border: 1px solid rgb(221, 219, 218);
        border-radius: 4px;
        background-clip: padding-box;
        height: 300px;
        min-width: 200px;
        box-shadow: 0 2px 2px 0 rgba(0, 0, 0, 0.10);
        .filter-card {
            .filter-card__header {
                display: flex;
                padding: 12px 16px 0 12px;
                .grid-icon {
                    display: flex;
                    align-items: center;
                    flex: 1 1 0%;
                    min-width: 0;
                    font-size: 14px;
                    .grid-icon-title {
                        // margin-left:24px;
                        flex: 2;
                        text-align: center;
                        font-size: 18px;
                    }
                }
            }
            .filter-card__body {
                padding: 12px 16px 12px 16px;
                .filter-card__body-search-key {
                    line-height: 28px;
                    margin-bottom: 20px;
                }
                .el-slider__button {
                    width: 10px;
                    height: 10px;
                }
                .sld-max-price {
                    margin-bottom: 20px;
                }
            }
        }
    }
</style>