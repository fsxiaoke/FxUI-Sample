## 简介

**FxUI-Sample-SPUExhibition**是一个开源的[纷享销客](https://www.fxiaoke.com/)自定义组件项目，它基于 [vue 2.x](https://github.com/vuejs/vue) 、 [FxUI](https://www.fxiaoke.com/mob/guide/uipaas/open/dist/index.html#/start/what)实现。后台数据通过[自定义函数](https://www.fxiaoke.com/mob/guide/crmdoc/src/8%E8%87%AA%E5%AE%9A%E4%B9%89%E5%87%BD%E6%95%B0.html)搭建。

商品列表信息不明显？无法突出重点？自定义页面可以重新定制化您的交互，设置自己的导航、分类等，可以配置出你想要的一眼能抓住重点的布局，您可以设置常用的字段单独更改并同步给详情，。



## 前序准备

>如果你是刚入门Vue的小白建议过一遍[Vue官方文档](https://cn.vuejs.org/),如果你已经掌握了Vue,那么强烈建议按照[Vue代码风格指南](https://cn.vuejs.org/v2/style-guide/)来写代码，[UI组件](https://www.fxiaoke.com/mob/guide/uipaas/open/dist/index.html#/component/ui/button)及[自定义函数](https://www.fxiaoke.com/mob/guide/crmdoc/src/8%E8%87%AA%E5%AE%9A%E4%B9%89%E5%87%BD%E6%95%B0.html)的使用也请您参考纷享文档。

此项目以【商品】对象为例，通过自定义页面重新规划出更能展示重点的“列表”。

## 完整项目图

## 客户字段配置
该项目以【商品】对象为例，您需要有以下几个字段：
>此配置并不唯一，您可以将自己的Api Name覆盖项目中的Api Name

字段名称 | Api Name
---|---
商品名称 | name
价格 | standard_price



## 自定义组上传
```
# 克隆项目到本地
git clone https://github.com/fsxiaoke/FxUI-Sample.git

```
登录到纷享销客并进入后台【管理】-CRM平台管理-【定制开发平台】-【自定义组件】-新建
- 名称：商品展列
- apiName：component_Ay4Nx__c（**apiName不固定**）
- 设置入口文件为**SPUExhibitionMain.vue**

## 自定义函数配置
#### 1、获取【商品】列表接口函数

##### 函数配置
- 配置自定义函数基本信息，配置如下：
  -  函数名称：**Get_List**
  -  Api Name：**Get_List__c**
  -  命名空间：**自定义控制器**
  -  返回值类型：**字典（Map）**
  -  绑定对象：**不绑定对象**
  -  描述：获取商品列表
- 自定义函数
  - 设置参数 (1)类型：String ,名称：apiname(2)类型：Map ,名称：search_query_info
 

```
    String ApiName = apiname
    
    Map searchQueryInfo = search_query_info as Map
    
    List filterData = searchQueryInfo.filters as List;
    
    Map filters = [:]
    log.info(filterData)
    filterData.each { item -> 
          Map currentFilter = item as Map
          if(currentFilter) {
            int objSize = currentFilter.keys().size() as Integer
            if(objSize) {
              List filedsValues = currentFilter.field_values as List
              switch(currentFilter.operator) {
              case "LIKE":
                currentFilter.value = Operator.LIKE(filedsValues[0])
                break;
              case "LTE":
                currentFilter.value = Operator.LTE(filedsValues[0])
                break;
              }
            
              filters.put(currentFilter.field_name,currentFilter.value)
            }
          }
      } 
    log.info(filters)
    List paramsFilter = [filters]
    int limit = searchQueryInfo['limit'] as Integer
    int offset = searchQueryInfo['offset'] as Integer
    def (Boolean error,QueryResult data,String errorMessage) =  Fx.object.find(ApiName,paramsFilter,limit,offset);
    
    if(error) {
        log.info("查询失败，原因：" + errorMessage)
    }else {
      return ["success":true,"data":data]
    }
```
#### 2、更新【商品】参数函数

##### 函数配置
- 配置自定义函数基本信息，配置如下：
  -  函数名称：**update_SPU_Info**
  -  Api Name：**update_SPU__c**
  -  命名空间：**自定义控制器**
  -  返回值类型：**字典（Map）**
  -  绑定对象：**不绑定对象**
  -  描述：更新商品字段
- 自定义函数
  - 设置参数 (1)类型：String ,名称：id(2)类型：String ,名称：apiname(3)类型：Map ,名称：update_data


```
    Map updateData = update_data as Map
    String CurrentID = id
    
    String CurrentApiName = apiname
    
    def (Boolean error,Map data,String errorMessage) =  Fx.object.update(CurrentApiName,id,updateData)
    
    if(error) {
        log.info("查询失败，原因：" + errorMessage)
    }else {
      return ["success":true]
    }
```
## 自定义页面配置
以上一切准备就绪就可以将组件配置到自定义页面了

登录到纷享销客并进入后台【管理】-CRM平台管理-【APP定制管理】-【CRM主页自定义】-【菜单模版管理】
编辑CRM，【添加自定义页面】，配置刚上传的自定义组件即可。