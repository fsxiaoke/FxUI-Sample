<template>
    <div class="all-wrapper" v-loading="dLoading">
        <div class="tip-info-wrapper" v-if="dShowPlane">
            <tip-container v-for="item in tipList" :key="item.type" :tipTitle="item.name" :tipType="item.type" :tipContent="item.content" :iconLink="item.iconLink" @triggerUpdate="assignHandle">
                <div slot="content">
                    <div v-if="item.showFlag" v-html="item.content"></div>
                    <div v-else-if="!item.showFlag && item.type === 'completeInfo'">
                        <complete-account :apiname="data.objectAPIName" :data="detailData" @cancel="cancelComplete" @updateDone="onUpdateDone" />
                    </div>
                </div>
            </tip-container>
        </div>
    </div>
</template>

<script>
import TipContainer from './TipContainer'
import CompleteAccount from './CompleteAccountMock'

// 不同场景的提示
const accountMap = {
    uncompleteInfo: `尚未完善，可<a href="javascript:void(0);"><strong>点此</strong></a>进行完善客户信息`,
    wrongcompleteInfo: `客户信息已同步, 您可以 <a href="javascript:void(0);"><strong>点此</strong></a>进行更新。
更新后需重新校验关联关系及受限情况。无法在天眼查找到此用户的英文名称信息，请手动维护。`,
    donecompleteInfo: `客户信息已同步, 您可以 <a href="javascript:void(0);"><strong>点此</strong></a>进行更新。
更新后需重新校验关联关系及受限情况。`,

    unvalidateInfo: `尚未进行校验，可<a href="javascript:void(0);"><strong>点此</strong></a>校验此客户是否为关联客户。`,
    wrongvalidateInfo: `已校验，此客户非关联客户。`,
    donevalidateInfo: `已校验，此客户为关联客户。`,

    unvalidateDemo: `尚未进行校验，可<a href="javascript:void(0);"><strong>点此</strong></a>校验可扩展示例(定制化逻辑)。`,
    wrongvalidateDemo: `已校验，可扩展示例(定制化逻辑)不存在。`,
    donevalidateDemo: `已校验，请检查“可扩展示例(定制化逻辑)”确认`
}
export default {
    name: 'InfoTemplate',
    components: {
        TipContainer,
        CompleteAccount
    },
    props: {
        data: Object
    },
    data() {
        return {
            dShowPlane: false,
            dLoading: true,
            detailData: {},
            tipList: [{
                name: '完善客户信息',
                type: 'completeInfo',
                completed: false,
                showFlag: true,
                apiname: 'Has_Improved_Acc_Info__c', //已完善客户信息
                relateApiname: 'English_Name__c' //客户英文名称
            }, {
                name: '关联客户校验',
                type: 'validateInfo',
                completed: false,
                showFlag: true,
                apiname: 'Has_Validate_Associated_Acc__c', //已校验关联客户
                relateApiname: 'Is_Associated_Account__c' //是否为关联客户
            }, {
                name: '可扩展示例（定制化逻辑）',
                type: 'validateDemo',
                completed: false,
                showFlag: true,
                apiname: 'Has_Validate_Demo__c',
                relateApiname: 'Is_Demo_Account__c'
            }],
            currentName: ''
        }
    },
    created() {
        this.fetchData()
    },
    methods: {
        fetchData() {
            FaciUI.objectApi.fetch_data(this.data.objectAPIName, this.data.objectId).then((data) => {
                this.dLoading = false
                this.detailData = data
                this.parseData()
            }).catch(err => {
                console.log(err)
            })
        },
        // 初始化显示页面
        parseData() {
            this.currentName = this.detailData['name']
            this.tipList.forEach(item => {
                item.completed = this.detailData[item.apiname]
                item.content = item.completed ? (this.detailData[item.relateApiname] ? accountMap[`done${item.type}`] : accountMap[`wrong${item.type}`]) : accountMap[`un${item.type}`]
                item.iconLink = item.completed ? 'https://a9.fspage.com/FSR/uipaas/FxUI-Sample-done.svg?v=1' : 'https://a9.fspage.com/FSR/uipaas/FxUI-Sample-undone.svg'
            })
            this.dShowPlane = true
        },
        assignHandle(type) {
            if (type === 'completeInfo') {
                this.tipList.forEach(item => {
                    if (item.type === 'completeInfo') {
                        item.showFlag = !item.showFlag
                    }
                })
            } else if (type === 'validateInfo') {
                if (!this.tipList[0]['completed']) {
                    this.$message({
                        message: '请先完善客户信息以获得正确的客户名称。',
                        type: 'warning'
                    })
                } else {
                    this.getPriceAuth()
                }
            } else {
                this.authValidateDemo()
            }
        },
        /**
         *  关联客户校验
         * */
        authValidateInfo(list) {
            let authFlag = Boolean(Array.isArray(list) && list.length)
            let updateData = {
                Is_Associated_Account__c: authFlag,
                Has_Validate_Associated_Acc__c: true,
                name: this.currentName
            }
            this.updateAccount(updateData, () => {
                this.dLoading = false
                this.tipList.forEach(item => {
                    if (item.type === 'validateInfo') {
                        item.content = authFlag ? '已校验，此客户为关联客户' : '已校验，此客户非关联客户'
                        item.completed = true
                        item.iconLink = 'https://a9.fspage.com/FSR/uipaas/FxUI-Sample-done.svg?v=1'
                    }
                })
            })
        },
        /**
         *  可扩展示例（定制化逻辑）校验
         * */
        authValidateDemo() {
            //do something...
        },
        onUpdateDone(obj) {
            obj.Has_Improved_Acc_Info__c = true
            this.detailData = obj
            this.parseData()
            this.cancelComplete()
            this.$emit('action', {
                type: 'refresh'
            })
        },
        /**
         * 查找 关联客户（示例对象，实际是名称为“关联客户”的关联客户的关联对象） 对象 是否有同名的客户
         * */
        getPriceAuth() {
            this.dLoading = true
            let objApiname = 'Associated_Account__c' // 关联客户apiname
            let url = `/EM1HNCRM/API/v1/object/${objApiname}/controller/List`

            let postData = {
                object_describe_api_name: objApiname,
                ignore_scene_record_type: false,
                search_query_info: `{"limit":1,"offset":0,"filters":[{"field_name":"name","field_values":["${this.detailData.name}"],"operator":"EQ"}],"orders":[{"fieldName":"last_modified_time","isAsc":false}]}`
            }

            PAAS.fly_api.post(url, postData).then((res) => {
                let list = res.Value ? res.Value.dataList : []
                this.authValidateInfo(list)
            })
        },
        /**
         * 更新数据
         */
        updateAccount(obj, cb) {
            let params = [{
                name: 'account_id',
                type: 'String',
                value: this.data.objectId
            }, {
                name: 'update_data',
                type: 'Map',
                value: obj
            }]
            // 批量新建face报价
            FaciUI.userDefine.call_controller('updateAccount__c', {
                params
            }).then(res => {
                console.log(res)
                if (res.Result.StatusCode === 0) {
                    cb()
                    this.dLoading = false
                } else {
                    this.$message({
                        message: res.Result.FailureMessage,
                        type: 'error'
                    })
                    this.dLoading = false
                }
            })
        },
        cancelComplete() {
            this.assignHandle('completeInfo')
        }
    }
}
</script>

<style>
    .tip-info-wrapper {
        display: flex;
        flex-wrap: wrap;
        flex-direction: column;
        align-items: center;
        align-content: center;
    }
</style>
