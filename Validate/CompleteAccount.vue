<template>
<div class="tianyancha-account-list">
    请确认您要选择的客户：

    <fx-table :data="tableData" tooltip-effect="dark" style="width: 100%" max-height="400" row-class-name="tableRowColor">
        <fx-table-column label="客户名称">
            <template scope="scope">
                <fx-radio :label="scope.row.name" v-model="radio" @change.native="getCurrentRow(scope.row)"></fx-radio>
            </template>
        </fx-table-column>
        <fx-table-column prop="legalPersonName" label="法定代表人" width="120">
        </fx-table-column>
        <fx-table-column prop="base" label="省份" width="120" show-overflow-tooltip>
        </fx-table-column>
    </fx-table>
    <div style="margin-top: 20px">
        <fx-button type="primary" @click="getAccountInfo">确定</fx-button>
        <fx-button @click="cancelHandler">取消</fx-button>
    </div>
</div>
</template>

<script>
export default {
    name: 'CompleteAccount',
    props: {
        apiname: String,
        data: Object
    },
    data() {
        return {
            radio: '',
            tableData: [],
            templateSelection: ''
        }
    },
    created() {
        this.fetchSearchData()
    },
    methods: {
        fetchSearchData() {
            let params = [{
                name: 'word',
                type: 'String',
                value: this.data.name
            }]
            FaciUI.userDefine.call_controller('func_daWet__c', {
                params
            }).then(res => {
                let controllerBack = res.Value.functionResult
                console.log(controllerBack)
                if (controllerBack) {
                    this.parseData(controllerBack.items)
                }
            })
        },
        parseData(dataList) {
            this.tableData = dataList.map(item => {
                return {
                    name: item.name.replace(/<em>|<\/em>/g, ''),
                    legalPersonName: item.legalPersonName,
                    base: item.base,
                    id: item.id
                }
            })
        },
        parseToCRM(data) {
            let updateData = {}
            updateData.name = data.name // name姓名(Name)
            updateData.TianYanCha_Id__c = data.id // id天眼查Id(TianYanCha_Id__c)-单行文本
            updateData.url = data.websiteList // websiteList网址(Website)
            updateData.tel = data.phoneNumber // phoneNumber电话(Phone)
            updateData.Industry__c = data.industry // industry行业(Industry)-单行文本
            updateData.remark = data.businessScope // businessScope描述(Description)
            updateData.Business_License__c = data.taxNumber // taxNumber营业执照号(Business_License__c)
            updateData.Registered_Capital_Str__c = data.regCapital // regCapital注册资本(Registered_Capital_Str__c)
            updateData.English_Name__c = data.property3 // 英文名
            updateData.TianYanCha_Url__c = `https://www.tianyancha.com/company/${data.id}` // 天眼查链接
            updateData.Has_Improved_Acc_Info__c = true
            this.updateAccount(updateData)
        },
        getCurrentRow(row) {
            this.templateSelection = row
        },
        /**
         * 获取所选公司具体信息
         */
        getAccountInfo() {
            let params = [{
                name: 'name',
                type: 'String',
                value: this.templateSelection.name
            }, {
                name: 'id',
                type: 'String',
                value: this.templateSelection.id
            }]
            FaciUI.userDefine.call_controller('func_k1ycD__c', {
                params
            }).then(res => {
                let controllerBack = res.Value.functionResult
                console.log(controllerBack)
                if (controllerBack) {
                    this.parseToCRM(controllerBack.result)
                }
            })
        },
        /**
         * 更新数据
         */
        updateAccount(obj) {
            let params = [{
                name: 'account_id',
                type: 'String',
                value: this.data._id
            }, {
                name: 'update_data',
                type: 'Map',
                value: obj
            }]
            FaciUI.userDefine.call_controller('updateAccount__c', {
                params
            }).then(res => {
                if (res.Value) {
                    if (res.Value.functionResult && res.Value.functionResult.errorMessage) {
                        this.$message({
                            message: res.Value.functionResult.errorMessage,
                            type: 'error'
                        })
                    } else {
                        this.$message({
                            message: '更新客户信息成功。',
                            type: 'success'
                        })
                        this.$emit('updateDone', this.data)
                    }
                }
            })
        },
        cancelHandler() {
            this.$emit('cancel')
        }
    }
}
</script>

<style>
    .tableRowColor {
        color: rgb(0, 109, 204);
    }

    .tianyancha-account-list .el-radio {
        color: rgb(0, 109, 204);
    }
</style>
