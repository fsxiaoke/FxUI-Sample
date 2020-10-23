## 简介

**FxUI-Sample-ValidateAccount**是一个开源的[纷享销客](https://www.fxiaoke.com/)自定义组件项目，它基于 [vue 2.x](https://github.com/vuejs/vue) 、 [FxUI](https://www.fxiaoke.com/mob/guide/uipaas/open/dist/index.html#/start/what)实现。后台数据通过[自定义函数](https://www.fxiaoke.com/mob/guide/crmdoc/src/8%E8%87%AA%E5%AE%9A%E4%B9%89%E5%87%BD%E6%95%B0.html)搭建。主要作用于纷享详情，利用天眼查查询关键字获取公司列表，并查询目标客户的营业执照号、注册资本、描述等信息并回填至纷享详情中。

您可以根据自己的场景来做逻辑处理，例如是否与其他对象（示例中为【关联对象】）中重复等

## 前序准备

>如果你是刚入门Vue的小白建议过一遍[Vue官方文档](https://cn.vuejs.org/),如果你已经掌握了Vue,那么强烈建议按照[Vue代码风格指南](https://cn.vuejs.org/v2/style-guide/)来写代码，[UI组件](https://www.fxiaoke.com/mob/guide/uipaas/open/dist/index.html#/component/ui/button)及[自定义函数](https://www.fxiaoke.com/mob/guide/crmdoc/src/8%E8%87%AA%E5%AE%9A%E4%B9%89%E5%87%BD%E6%95%B0.html)的使用也请您参考纷享文档。

上传的自定义组件中仅支持vue,js,less,scss,css类型文件，且**以下的API Name名称、函数名称等并非固定值，您可以使用自己的API Name或函数名，如若您使用的是自己的定义的名称，请在自定义函数或vue文件中留意调用函数名称等API Name是否与您设置的一致。**
## 完整项目图

## 客户字段配置（配置客户对象下需回填的字段）

>您可以在**CompleteAccount.vue文件下**的**parseToCRM**方法中配置你需要回填的Api Name,此处的Api Name并不唯一，您可以设置您自己的Api Name，只需在使用时统一即可。

**以下类型皆为单行文本**

TianYanCha_Id__c: 天眼查ID

Industry__c: 天眼查行业

Business_License__c: 营业执照号

Registered_Capital_Str__c: 注册资本

English_Name__c: 客户英文名称

TianYanCha_Url__c: 天眼查链接


**以下类型皆为布尔值,选项设置【否】**

Has_Improved_Acc_Info__c: 已完善客户信息

Has_Validate_AssociatedAcc__c: 已校验关联客户

Is_Associated_Account__c: 是否为关联客户
## 自定义组上传

```
# 克隆项目到本地
git clone https://github.com/fsxiaoke/FxUI-Sample.git

```
登录到纷享销客并进入后台【管理】-CRM平台管理-【定制开发平台】-【自定义组件】-新建
- 名称：天眼查客户验证
- apiName：validateAccountTwo__c（**apiName不固定，更改后需替换所有使用处**）
- 设置入口文件
  - 若您想使用mock数据查看示例使用**ValidateMainMock.vue**作为入口文件
  - 若您有天眼查的**Authorization**，使用**ValidateMain.vue**作为入口文件（**自定义函数配置**中会解释为何要分两个版本）


## 自定义组件配置
>您可以选择将组件放置您需要的且自定义组件所支持的任意位置，本示例以客户下自定义按钮的UIAction为例，配置方式并不唯一。

##### 在客户下创建一个名为【校验客户】的自定义按钮
- 1、设置按钮类型为UI按钮。
- 2、基本信息中设置按钮名称为**校验客户**，API Name设置为**button_47u9x__c**（此处的button_47u9x__c并非固定值，您可以使用自己的API Name，如若您使用的是自己的API Name，请在自定义函数使用过程中留意API Name是否与您设置的API Name一致）。
- 3、选择按钮位置为**详情页**
- 4、添加用户点击按钮后触发的操作，配置如下：
  - 函数名称：**Acc_ValidateButton**
  - Api Name：**func_0865b__c**
  - 命名空间：**按钮**
  - 返回值类型：***UIAction**
  - 绑定对象：**客户**
  - 描述：带出客户校验UI界面
- 5、自定义函数为：

```
    UIAction action = OpenDialogAction.build() {
      title = "校验客户"
      width = 800
      component {
        apiName = 'validateAccountTwo__c'//validateAccountTwo__c为您上传的自定义组件的Api Name
      }
    }
    
    return action

```




## 自定义函数配置
#### 1、调用天眼查接口函数
>！请注意，示例中调用天眼查的接口的Authorization为虚拟值，您可以到[天眼查](https://open.tianyancha.com/open/816)申请您需要的接口，使用您自己的Authorization。若果您只想体验也没有关系，我们准备了mock数据,您只需要在上传组件中上传mock版本的vue文件，我们设置了查询参数为**纷享销客**的数据，且您可以选择第一个数据进行回填，**若您选择mock版本，可以忽略此步骤直接跳转到更新客户信息函数继续处理**。
##### (1)根据关键字查询列表
##### 函数配置
- 配置自定义函数基本信息，配置如下：
  -  函数名称：**Acc_TYCQuery**
  -  Api Name：**func_daWet__c**
  -  命名空间：**自定义控制器**
  -  返回值类型：**字典（Map）**
  -  绑定对象：**不绑定对象**
  -  描述：天眼查列表查询函数
- 自定义函数
  - 设置参数 (1)类型：String ,名称：word
 

```
String aname=word as String;
String url="https://open.api.tianyancha.com/services/open/search/2.0?word="+aname;

def(Boolean Error, HttpResult OutData, String ErrorMessage) = Fx.http.get(url,["Authorization":"1111aaaa-1111-1111-1111-111111111111"],2000,true,2);
def rstData=OutData['content'] ;
def items=rstData['result']['items'] 
Map  result=["items":items]
return result


```

##### (2)查询详细信息
##### 函数配置
- 配置自定义函数基本信息，配置如下：
  -  函数名称：**Acc_TYCDetail**
  -  Api Name：**func_k1ycD__c**
  -  命名空间：**自定义控制器**
  -  返回值类型：**字典（Map）**
  -  绑定对象：**不绑定对象**
  -  描述：天眼查详情查询函数
- 自定义函数
  - 设置参数 (1)类型：String ,名称：name
  (2)类型：String ,名称：id
```
String aname=name as String;
String  tid = id as String;
String url="https://open.api.tianyancha.com/services/open/ic/baseinfoV2/2.0?id="+tid+"&name="+aname;

def(Boolean Error, HttpResult OutData, String ErrorMessage) = Fx.http.get(url,["Authorization":"11111111-1111-1111-1111-111111111111"],2000,true,2);

if(OutData) {
 def rstData=OutData['content'] ;
 def items=rstData['result']
 Map  result=["result":items]
return result 
}
```
#### 2、更新客户信息函数
##### 函数配置
- 配置自定义函数基本信息，配置如下：
  -  函数名称：**updateAccountInfo**
  -  Api Name：**updateAccount__c**
  -  命名空间：**自定义控制器**
  -  返回值类型：**字典（Map）**
  -  绑定对象：**不绑定对象**
  -  描述：更新客户信息
- 自定义函数
  - 设置参数（1）类型：Map ,名称：update_data（2）类型：String ,名称：account_id
>！请注意，示例中调用天眼查的接口的Authorization为虚拟值，您可以到[天眼查](https://open.tianyancha.com/open/818)申请您需要的接口，使用您自己的Authorization
```
Map updateData = update_data as Map
String CurrentID = account_id

String AccountName = updateData["name"]

def (Boolean receivableError,QueryResult receivableResult,String receivableErrorMessage) =  Fx.object.find("AccountObj",[["name":AccountName]],2,0)
if(receivableError) {
    log.info("查询失败，原因：" + receivableErrorMessage)
}else {
    log.info('查询成功' + receivableResult)

    List receivableList = receivableResult.dataList
    Map receivable = receivableList[0] as Map
    if(receivableList.size() > 1 || (receivable && receivable['_id'] != CurrentID)) {
        return ["errorMessage":"已存在重名客户，无法更新。"]
    }else{
      //增加敏感词检查
      if(AccountName)
        {
          def ret = Fx.proxy.callAPI("dirtywords.megvii", null,["text":AccountName])
          log.info(ret)
          if(!ret[0]){
            def content = ret[1]['content'] as List
            log.info(content)
            if( content.size()>0 )
            {
                def keyword = content[0]['keyword']
                log.info('is dirty word '+keyword)
                String dirtyword="包含敏感词"+keyword;
                  updateData.remove("name")
                  log.info("更新参数"+updateData)
                  def(Boolean error,Map data,String errorMessage) = Fx.object.update("AccountObj",CurrentID,updateData)

                return ["errorMessage":dirtyword] 
            }else
            { 
                log.info("更新参数"+updateData)
                def(Boolean error,Map data,String errorMessage) = Fx.object.update("AccountObj",CurrentID,updateData)
                if(error) {
                  log.info("更新失败，原因：" + errorMessage)
                }else {
                  log.info('更新成功' + data)
                  return ["success":true]
                }
            }
         }
      }
    }
}
```

