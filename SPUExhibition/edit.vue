<template>
    <faci-widget-frame :debounce="100" class="sample-product-edit" mode="wide">
        <div class="sample-product-edit__wrapper">
            <header class="sample-product-edit__header">
                <div class="grid-icon">
                    <i class="el-icon-s-order"></i>
                    <div class="grid-icon-title">{{detailData.name}}</div>
                </div>
                <faci-field-display 
                    :field="field" 
                    :value="value"
                    valueStyle="line-height: 18px;width:56px;padding-left:4px;"
                    labelStyle="display:none;"></faci-field-display>
                <div class="close" @click="closeEditCard">x</div>
            </header>
            <div class="sample-product-container">
                <img :src="`${getFscLink(detailData.picture)[0]}`" alt="" class="sample-product-container__show">
                <div class="sample-product-container__form">
                    <div class="sample-product-form__item">
                        <div class="sample-product-form__label">商品名称</div>
                        <fx-input 
                            v-model="formData.name" 
                            placeholder="请输入内容"
                            class="sample-product-form__input" 
                            size="mini"
                            :debounce="200"></fx-input>
                    </div>
                    <div class="sample-product-form__item">
                        <div class="sample-product-form__label">标准价格</div>
                        <fx-input 
                            v-model="formData.standard_price" 
                            placeholder="请输入内容" 
                            class="sample-product-form__input" 
                            size="mini"
                            :debounce="200"></fx-input>
                    </div>
                </div>
                <fx-button type="primary" @click="save" class="primary-button">保存</fx-button>
            </div>
        </div>
    </faci-widget-frame>
</template>
<script>

export default {
    props: ['detailData'],
    data() {
        return {
            formData: {},
            field: {
                type: 'object_reference',
                api_name: 'cus_field',
                target_api_name: 'SPUObj'
            },
            value: {
                cus_field: this.detailData._id,
                cus_field__r: '查看详情'
            }
        }
    },
    created() {
        this.formData = Object.assign(this.formData, {
            name: this.detailData.name,
            standard_price: this.detailData.standard_price
        })
    },
    watch: {
        detailData(cur, old) {
            this.formData = Object.assign(this.formData, {
                name: this.detailData.name,
                standard_price: this.detailData.standard_price
            })
            this.$set(this.value,'cus_field',cur._id)
        }
    },
    methods:{
        getFscLink (data) {
            if(data && data.length > 0) {
                return data.map((item) => {
                    return CRM.util.getFscLink(`${item.path}.${item.ext}`)
                })
            }else{
                return ['https://a9.fspage.com/FSR/uipaas/empty-image.svg']
            }
        },
        save () {
            let me = this;
            let params = [{
                name: 'id',
                type: 'String',
                value: this.detailData._id
            }, {
                name: 'apiname',
                type: 'String',
                value: 'SPUObj'
            },{
                name: 'update_data',
                type: 'Map',
                value: this.formData
            }];
            FaciUI.userDefine.call_controller('update_SPU__c', {
                params
            }).then(res => {
                if(res.Value.functionResult && res.Value.functionResult.success){
                    me.$emit('refresh')
                }else if(res.Value.functionResult && res.Value.functionResult.errorMessage) {
                    this.$message({
                        message: res.Value.functionResult.errorMessage,
                        type: 'error'
                    })
                }
            })
        },
        closeEditCard() {
            this.$emit('closeEditCard')
        }
    }
}
</script>
<style lang="less">
    .sample-product-edit {
        margin: 0 12px;
    }
    .sample-product-edit__wrapper {
        font-size: 14px;
        line-height: 14px;
        position: relative;
        display: flex;
        padding:0 12px 12px 12px;
        background: rgb(255, 255, 255);
        border: 1px solid rgb(221, 219, 218);
        border-radius: 4px;
        background-clip: padding-box;
        min-height: 300px;
        display: flex;
        flex-wrap: wrap;
        align-items: flex-start;
        box-shadow: 0 2px 2px 0 rgba(0, 0, 0, 0.10);
        .sample-product-edit__header{
            display: flex;
            padding: 12px 16px 0 12px;
            margin-bottom: 12px;
            align-items: center;
            width: 100%;
            .grid-icon {
                display: flex;
                align-items: center;
                flex: 1 1 0%;
                font-size: 18px;
                .grid-icon-title {
                    flex:2;
                    text-align: center;
                    font-size: 18px;
                    line-height: 18px;
                }
            }
            .close {
                position: absolute;
                right: 8px;
                top: 6px;
                cursor: pointer;
            }
        }
        .sample-product-container{
            .sample-product-container__show{
                position: relative;
                max-width: 100%;
                max-height: auto;
            }
            .sample-product-container__form{
                .sample-product-form__label {
                    padding: 5px 0;
                }
                .sample-product-form__item {
                    line-height: 28px;
                    margin-bottom: 5px;
                }
            }
            .el-button.el-button--primary.primary-button {
                width: 100%;
                margin-top: 18px;
            }
        }
    }
</style>