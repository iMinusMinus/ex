# 数据交换

## 数据交换格式
|    |protobuf|hessian|native|xml|json|
|:---|:-------|:------|:-----|:--|:---|
|速度|        |       |       |   |    |
|大小|        |       |      |   |     |
|跨语言|      |       |      |    |    |

## JSON
|     |Jackson(com.fasterxml.jackson) |FastJson(com.alibaba.fastjson)|Gson(com.google.gson)|
|:----|:-------|:-----|:-------|
|注释 |JsonParser.Feature#ALLOW_COMMENTS|Feature#AllowComment|built-in|
|单引号|JsonParser.Feature#ALLOW_SINGLE_QUOTES|Feature#AllowSingleQuotes,SerializerFeature#UseSingleQuotes|built-in|
|泛型 |TypeReference|TypeReference|TypeToken|
|多态 |@JsonTypeInfo(use=)|SerializerFeature#WriteClassName|         |
|带参构造函数|@JsonCreator & @JsonProperty(value=)|@JSONCreator & @JSONField(name=)|built-in|
|引用|      |"$ref":"$.path"||
|别名|@JsonProperty(value=) or @JsonNaming|@JSONField(name=)|@SerializedName(alternate=)|
|日期格式|ISO8601|ISO8601|ISO8601|
|自定义|JsonSerializer、JsonDeserializer、@JsonSerialize、@JsonDeserialize、TypeResolverBuilder、@JsonTypeResolver|       |@JsonAdapter、TypeAdapter、JsonSerializer、JsonDeserializer|

_Jackson支持多种形式标识多态类型，字段可以是'@class'、'@type'，同一个包下甚至可以使用'@c'。同时也支持指定字段。
字段可以作为整个对象的一部分，也可以是以类型为key，对象值为value。或者类型与对象组成数组，类型在前。   
Jackson可以通过设置ObjectMapper.setPropertyNamingStrategy配置别名映射。   
Jackson可通过ObjectMapper.setDateFormat修改默认的日期格式。_

_FastJson使用'@type'来标识类型，可以使用com.alibaba.fastjson.JSON#setDefaultTypeKey修改。多态时类型标识必须最先出现_

_Gson可以利用GsonBuilder.setFieldNamingStrategy来设置别名处理。   
Gson利用Unsafe来初始化对象，不需要特殊标识就可以处理带参构造函数的反序列化。   
可使用GsonGsonBuilder.setDateFormat来设置默认的日期格式。   
Gson对多态支持不友好，需要注册并实现com.google.gson.TypeAdapter_
