### API接收地址

```
https://nb-pipeline.qiniuapi.com
```

### API返回内容

**响应报文** 

* 如果请求成功,返回HTTP状态码`200`:

```
HTTP/1.1 200 OK
```

* 如果请求失败,返回包含如下内容的JSON字符串（已格式化,便于阅读）:

```
{
    "error":   "<errMsg    string>"
}
```

* 如果请求包含数据获取,则返回相应数据的JSON字符串；


### 创建消息队列(数据源)


**请求语法**

```
POST /v2/repos/<RepoName>
Content-Type: application/json
Authorization: Pandora <auth>
{
    "region": <Region>,
    "schema": [
      {
        "key": <Key>,
        "valtype": <ValueType>,
        "elemtype": <ElemType>,
        "required": <Required>,
        "schema": [
            ...
        ]
      },
      ...
    ],
    "options":{
      "withIP":<ipkeyname>
    }
}
```

**请求内容**

|参数|类型|必填|说明|
|:---|:---|:---:|:---|
|RepoName|string|是|消息队列名称</br>命名规则: `^[a-zA-Z_][a-zA-Z0-9_]{0,127}$`|
|region|string|是|计算与存储所使用的物理资源所在区域</br>目前仅支持“nb”(华东区域)|
|schema|array|是|数据的字段信息</br>由‘字段名称’、‘字段类型’、‘数组类型’、‘是否必填’组成</br>|
| schema.key|string|是|字段名称</br>命名规则: `^[a-zA-Z_][a-zA-Z0-9_]{0,127}$`|
| schema.valtype|string|是|字段类型</br>目前仅支持：</br>`boolean`：布尔类型</br>`long`：整型</br>`date`：RFC3339日期格式</br>`float`：64位精度浮点型</br>`string`：字符串</br>`array`：数组</br>`map`：嵌套类型，可嵌套，最多5层，类似于json object</br>`jsonstring`：符合json格式的字符串</br>|
| schema.elemtype|string|否|数组类型</br>当`schema.valtype:"array"`时必填</br>目前仅支持`long`、`float`、`string`|
| schema.required|bool|否|是否必填</br>用户在传输数据时`key`字段是否必填|
|options|map|否|表达一些repo的可选项|
|options.withIP|string|否|在写入的数据中加入用户的来源IP信息，并命名为<ipkeyname>字段，加入到schema中，类型为string；若命名为空则不加入。|

**示例**

```
curl -X POST https://nb-pipeline.qiniuapi.com/v2/repos/Test_Repo \
-H 'Content-Type: application/json' \
-H 'Authorization: Pandora 2J1e7iG13J66GA8vWBzZdF-UR_d1MF-kacOdUUS4:NTi3wH_WlGxYOnXsvgUrO4XMD6Y='  \
-d '{
    "region": "nb", 
    "schema": [
        {
            "key": "userName",
            "valtype": "string",
            "required": true
        },
        {
            "key": "age",
            "valtype": "float",
            "required": true
        },
        {
            "key": "addresses",
            "valtype": "array",
            "elemtype": "long",
            "required": true
        },
        {
            "key": "profile",
            "valtype": "map",
            "required": true,
            "schema": [
              {
                  "key": "position",
                  "valtype": "string",
                  "required": true
              },
              {
                  "key": "salary",
                  "valtype": "float",
                  "required": true
              },
              {
                  "key": "education",
                  "valtype": "array",
                  "elemtype": "string",
                  "required": false
              }
            ]
        }
    ]
}'
```

### 查看所有消息队列

**请求语法**

```
GET /v2/repos
Authorization: Pandora <auth>
```

**响应报文** 

```
Content-Type: application/json
{
    "repos": [
      {
        "name": <RepoName>,
        "region": <Region>,
        "derivedFrom": <TransformName>
      },
      ...
    ]
}
```

**响应内容**

|参数|类型|必填|说明|
|:---|:---|:---:|:---|
|derivedFrom|string|-|表示这个消息队列是由哪个transform生成的</br>如果此项为空,说明该消息队列是由用户自行创建的|



### 根据名称查询消息队列

**请求语法**

```
GET /v2/repos/<RepoName>
Authorization: Pandora <auth>
```

**响应报文** 

```
Content-Type: application/json
{
    "region": <Region>,
    "derivedFrom": <TransformName>,
    "schema": [
      {
        "key": <Key>,
        "valtype": <ValueType>,
        "elemtype": <ElemType>,
        "required": <Required>,
        "schema": [
            ...
        ]
      },
      ...
    ]
}
```

### 根据名称删除消息队列

**请求语法**

```
DELETE /v2/repos/<RepoName>
Authorization: Pandora <auth>
```

### 根据名称更新消息队列

**请求语法**

```
PUT /v2/repos/<RepoName>
Content-Type: application/json
Authorization: Pandora <auth>

{
	"schema": [
      {
        "key": <Key>,
        "valtype": <ValueType>,
        "elemtype": <ElemType>,
        "required": <Required>,
        "schema": [
            ...
        ]
      },
      ...
    ]
}
```

!> 注意： 更新字段信息时，如果需要保留已有的字段信息，也需要填写上去，这是一次全量更新。

### 创建流式计算任务

**请求语法**

```
POST /v2/repos/<RepoName>/transforms/<TransformName>/to/<DestinationRepoName>
Content-Type: application/json
Authorization: Pandora <auth>
{
  "plugin": {
    "name": <PluginName>,
    "output": [
      {
        "name": <FieldName1>,
        "type": <FieldType>
      },
      {
        "name": <FieldName2>
      },
      ......
    ],
  },
  "mode": <Mode>,
  "code": <Code>,
  "interval": <Interval>,
  "container": {
       "type": <ContainerType>,
       "count": <ContainerCount>
  },
  "whence": <TransformWhence>,
  "destrepo": [
      {
        "key": <Key>,
        "valtype": <ValueType>,
        "elemtype": <ElemType>,
        "required": <Required>,
        "schema": [
            ...
        ]
      },
      ...
  ]

}
```

**请求内容**

|参数|类型|必填|说明|
|:---|:---|:---:|:---|
| RepoName |string|是|指定一个消息队列的名称|
| TransformName |string|是|计算任务名称</br>用来标识该消息队列的唯一性</br>命名规则: `^[a-zA-Z_][a-zA-Z0-9_]{0,127}$`|
| DestinationRepoName |string|是|计算结果输出消息队列</br>如果该消息队列不存在</br>将自动创建一个|
| plugin |json|否|自定义计算|
| name |string|是|自定义计算名称|
| output |json|是|输出数组</br>即这个plugin计算完成后，输出的数据结果的结构和类型</br>也可以理解为一张表,包含字段名称和字段类型|
|output.name|string|是|输出字段名称</br>命名规则: `^[a-zA-Z_][a-zA-Z0-9_]{0,127}$`|
|output.type|string|否|输出字段类型</br>支持`string`、`long`和`float`三种</br>时间类型使用`long`类型</br>默认为`string`类型|
| mode |string|否|该计算任务使用的语言类型</br>目前仅支持`sql`|
| code |string|否|`sql`语句代码|
| interval |string|否|计算任务的运行时间间隔</br>目前支持`5s`、`10s`、`20s`、`30s`、</br>`1m`、`5m`和`10m`的粒度</br>如果不指定，系统默认使用`1m`|
|container|map|否|计算资源的数量及类型|
|type|string|否|目前支持`1U2G`、`1U4G`、`2U4G`、`4U8G`、`4U16G`和`8U16G`</br>分别代表</br>`1核(CPU)2G(内存)`、`1核(CPU)4G(内存)`、`2核(CPU)4G(内存)`、`4核(CPU)8G(内存)`、`4核(CPU)16G(内存)`、`8核(CPU)16G(内存)`|
|count|int|否|指资源`type`的数量,最小为1,没有上限|
|whence|string|否|计算数据的起始位置, 目前支持oldest、newest, 分别表示从指定仓库的最早、最新数据开始计算, 默认值为newest|

!> 注意:`mode`加`code`是基础的数据计算方式,自定义计算(plugin)是更为高级的数据计算方式,要注意`mode/code`和自定义计算两种计算方式可以共存,但不可以一种都不指定。当自定义计算和`mode/code`共存时,系统优先执行自定义计算,后执行mode/code。



**示例**

```
curl -X POST https://nb-pipeline.qiniuapi.com/v2/repos/test_repo/transforms/transform_job/to/compute_repo \
	 -H 'Content-Type: application/json' \
	 -H 'Authorization: Pandora 2J1e7iG13J66GA8vWBzZdF-UR_d1MF-kacOdUUS4:NTi3wH_WlGxYOnXsvgUrO4XMD6Y=' \
	 -d {
  		"mode": "sql",
  		"code": "select count(*) from test_repo",
  		"interval": "1m",
  		"container": {
       		"type": "M16C4",
       		"count": 5
  			}
		}
```


### 上传jar包(自定义计算使用)

**请求语法**

```
POST /v2/plugins/<PluginName>
Content-Type: application/java-archive
Content-MD5: <ContentMD5>
Authorization: Pandora <auth>
```

**请求内容**

|参数|类型|必填|说明|
|:---|:---|:---|:---|
|PluginName|string|是|plugin名称</br>命名规则: `^[a-zA-Z][a-zA-Z0-9_\\.]{0,127}[a-zA-Z0-9_]$`|
|ContentMD5|string|是|jar包的MD5码|

**Plugin说明:**

* Jar包的命名必须和包含代码方法的类名一致
* 上传的Plugin Jar包最大为100MB。
* Content-MD5头部是可选的。如果上传plugin的时候带上该头部服务器会校验上传数据的校验和,如果两者不一致服务器将拒绝上传。如果不带该头部,服务器不做任何校验和的检查。
* <ContentMD5>是先计算plugin内容的MD5,再对MD5做一次base64编码转化为字符串。例如qiniu这个字符串的Content-MD5是gLL29S04bTCxYd2kCqsEIQ==而不是7b9d6b4d89f6825a196d4cc50fdbedc5
* PluginName必须与用户所编写的Parser类的全限定名保持一致,否则transform执行plugin会失败。 例如NginxLogParser位于com.qiniu包,PluginName须写为com.qiniu.NginxLogParser。

**示例**

```
curl -X POST https://nb-pipeline.qiniuapi.com/v2/plugins/ComputeSumDataParser \
-H 'Content-Type: application/java-archive'  \
-H 'Content-MD5: 900150983cd24fb0d6963f7d28e17f72'  \
-H 'Authorization: Pandora 2J1e7iG13J66GA8vWBzZdF-UR_d1MF-kacOdUUS4:NTi3wH_WlGxYOnXsvgUrO4XMD6Y=' \
-T ./TestPlugin.jar \  
```

### 查看所有流式计算任务

**请求语法**

```
GET /v2/repos/<RepoName>/transforms
Authorization: Pandora <auth>
```

### 查看指定流式计算任务的信息

**请求语法**

```
GET /v2/repos/<RepoName>/transforms/<TransformName>
Authorization: Pandora <auth>
```

**响应报文** 

```
200 OK
Transform-Type: application/<TransformType>
{
    "name": "<TransformName1>",
    "to": "<DestRepo1>",
    "spec": {
      "plugin": {
        "name": <PluginName>,
        "output": [
          {
            "name": <FieldName1>,
            "type": <FieldType>
          },
          {
            "name": <FieldName2>
          },
          ...
        ],
      },
      "mode": <Mode>,
      "code": <Code>,
      "interval": <Interval>
    }
}
```


### 修改流式计算任务

**请求语法**

```
PUT /v2/repos/<RepoName>/transforms/<TransformName>
Content-Type: application/json
Authorization: Pandora <auth>
{
	"plugin": {
      "name": <PluginName>,
      "output": [
        {
          "name": <FieldName1>,
          "type": <FieldType>
        },
        {
          "name": <FieldName2>
        },
        ...
      ],
    },
    "code": <Code>,
    "interval": <Interval>,
    "container": {
      "type":  <ContainerType>,
      "count": <ContainerCount>
    }
}
```



>**注意:** 
>
>1.更新计算任务,可以同时更新`plugin`、`sql代码`、`运行时间间隔`和`配额`,也可以只更新其中一种,但不能一种都不指定。

>2.在更新`sql`和`plugin的输出字段`时,只能添加新的字段,不能删除和更改已经存在的字段。

### 删除流式计算任务

**请求语法**

```
DELETE /v2/repos/<RepoName>/transforms/<TransformName>
Authorization: Pandora <auth>
```

### 流式导出数据至HTTP地址

**请求语法**

```
POST /v2/repos/<RepoName>/exports/<ExportName>
Content-Type: application/json
Authorization: Pandora <auth>
{
  "type": <http>,
  "whence": <ExportWhence>,
  "spec": {
		"host": <Host>,      
		"uri": <RequestURI>,
		"format": <ExportFormat>  
	}
}
```

**请求内容**

|参数|类型|必填|说明|
|:---|:---|:---:|:---|
| RepoName |string|是|需要导出数据的消息队列名称|
| ExportName |string|是|导出任务名称</br>命名规则: `^[a-zA-Z_][a-zA-Z0-9_]{0,127}$`|
| Type |string|是|导出方式</br>目前支持`http`、`logdb`、`mongo`、`tsdb`、`kodo`、`report`</br>在这里我们选择`http`|
|whence|string|否|导出数据的起始位置</br>目前支持`oldest`、`newest`,</br>分别表示从指定仓库的`最早`、`最新`数据开始导出</br>默认值为oldest|
| Spec |json|是|导出任务的参数主体</br>选择不同的`type`</br>`Spec`也需要填写不同的参数</br>将在下面分开讲解|
| host |string|是|服务器地址（ip或域名）</br>例如:`https://nb-pipeline.qiniuapi.com` </br>或 `127.0.0.1:7758`|
| uri |string|是|请求资源路径（具体地址,不包含ip或域名）</br>例如:`/test/repos`|
| format |string|否|导出方式</br>支持`text`和`json`</br>如果没有填写此项，默认为`text`|

!> 注意: 导出数据格式和`推送数据`相同。

**示例**

```
curl -X POST https://nb-pipeline.qiniuapi.com/v2/repos/test_Repo/exports/export_job1 \
-H 'Content-Type: application/json' \
-H 'Authorization: Pandora 2J1e7iG13J66GA8vWBzZdF-UR_d1MF-kacOdUUS4:NTi3wH_WlGxYOnXsvgUrO4XMD6Y=' \
-d '{
  "type": "http",
  "spec": {
	"host": "www.qiniu.com",
	"uri": "/test/repos"
  }
}'
```

### 流式导出数据至时序数据库

**请求语法**

```
POST /v2/repos/<RepoName>/exports/<ExportName>
Content-Type: application/json
Authorization: Pandora <auth>
{
  "type": <tsdb>,
  "whence": <ExportWhence>,
  "spec": {
        "destRepoName": <DestRepoName>,
        "series": <SeriesName>,
        "omitInvalid": <OmitInvalid>,
        "omitEmpty": <OmitEmpty>,
        "tags": {
            "tag1": <#key1>,
            "tag2": <#key2>,
            ...
        },
        "fields": {
            "field1": <#key1>,
            "field2": <#key2>,
            ...
        },
        "timestamp": <#key1>,   
     }
}
```
**请求内容**

|参数|类型|必填|说明|
|:---|:---|:---:|:---|
| destRepoName |string|是|数据库名称|
| series |string|是|序列名称|
| omitInvalid |bool|否|是否忽略无效数据，默认值为false|
| omitEmpty |bool|否|当某条数据的字段取值全部为null时是否忽略该条数据，默认值为false，设置为true时可避免导出没有意义的数据|
| tags |map|是|索引字段|
| fields |map|是|普通字段|
| timestamp |string|否|时间戳字段</br>会用rfc3339日期格式进行解析</br>如果格式不正确则会抛弃这一条数据</br>如果此项为空，则默认使用当前时间|

> 时序数据库中的timestamp字段的类型必须为 date；
> 消息队列中字段类型为:Long/String/Date 的字段都可以导出至时序数据库中的 timestamp 字段

**示例**

```
curl -X POST https://nb-pipeline.qiniuapi.com/v2/repos/test_Repo/exports/export_job4 \
-H 'Content-Type: application/json' \
-H 'Authorization: Pandora 2J1e7iG13J66GA8vWBzZdF-UR_d1MF-kacOdUUS4:NTi3wH_WlGxYOnXsvgUrO4XMD6Y=' \
-d '{
  "type": "tsdb",
  "spec":{
      	"destRepoName": "test_tsdb",
		"series": "req_io",
		"tags": {"type": "#type","src": "#src","zone": "#zone","time": "#time","bucket": "#bucket","domain": "#domain"
		},
		"fields": {"hits": "#hits","flow": "#flow"},            
	}
}'
```

### 流式导出数据至日志检索服务

**请求语法**

```
POST /v2/repos/<RepoName>/exports/<ExportName>
Content-Type: application/json
Authorization: Pandora <auth>
{
  "type": <logdb>,
  "whence": <ExportWhence>,
  "spec": {
        "destRepoName": <DestRepoName>,
        "omitInvalid": <OmitInvalid>,
        "omitEmpty": <OmitEmpty>,
        "doc": {
            "toRepoSchema1": <#fromRepoSchema1>,
            "toRepoSchema2": {
            	"toRepoSchema3": <#fromRepoSchema3>,
            }
            ......
        }
}
```

**请求内容**

|参数|类型|必填|说明|
|:---|:---|:---:|:---|
| destRepoName |string|是|日志仓库名称|
| omitInvalid |bool|否|是否忽略无效数据，默认值为false|
| omitEmpty |bool|否|当某条数据的字段取值全部为null时是否忽略该条数据，默认值为false，设置为true时可避免导出没有意义的数据|
| doc |map|是|字段关系说明</br> `fromRepoSchema`表示源消息队列字段名称</br>`toRepoSchema`表示目标日志仓库字段名称|

> 消息队列中,字段的类型与日志检索服务中的字段类型需要作出如下对应:
> 
> 消息队列类型:string 对应 日志检索服务:string / date
> 
> 消息队列类型:long 对应 日志检索服务:long / date
> 
> 消息队列类型:float 对应 日志检索服务:float
> 
> 消息队列类型:array[string] 对应 日志检索服务:string
> 
> 消息队列类型:array[long] 对应 日志检索服务:long
> 
> 消息队列类型:array[float] 对应 日志检索服务:float
> 
> 消息队列类型:map 对应 日志检索服务:object
> 
> 消息队列类型:date 对应 日志检索服务:date 
> 
> 消息队列类型:jsonstring 对应 日志检索服务:object

!> 注意: 对于消息队列的`jsonstring`类型，导出至日志检索服务的`object`类型时，会将内嵌字段名称中所有圆点(`.`)替换为下划线（由于日志检索服务的object类型的内嵌字段名称不支持圆点(`.`))。例如，消息队列中某一个字段`f1`的类型为`jsonstring`，取值为`{"education.level": "university"}`，最终导出至日志检索服务为`{"education_level": "university"}`，内嵌字段`education.level`中的圆点(`,`)被替换为下划线(`_`)。

**示例**

```
curl -X POST https://nb-pipeline.qiniuapi.com/v2/repos/test_Repo/exports/export_job2 \
-H 'Content-Type: application/json' \
-H 'Authorization: Pandora 2J1e7iG13J66GA8vWBzZdF-UR_d1MF-kacOdUUS4:NTi3wH_WlGxYOnXsvgUrO4XMD6Y=' \
-d '{
  "type": "logdb",
  "spec": {
	  "destRepoName": "logdb_testRepo", 
	  "doc":{
			"user":"userName",
			"profile":{
				"age":"age"
			}
	  }
	}
}'
```

### 流式导出数据至对象存储服务

**请求语法**

```
POST /v2/repos/<RepoName>/exports/<ExportName>
Content-Type: application/json
Authorization: Pandora <auth>
{
    "type": <kodo>,
    "whence": <ExportWhence>,
    "spec": {
         "bucket": <Bucket>,
         "keyPrefix": <KeyPrefix>, 
         "email": <Email>,  
         "accessKey": <AccessKey>,    
         "fields": {
             "key1": <#value1>,
             "key2": <#value2>,
             ...
         },
         "rotateStrategy": <RotateStrategy>,
         "rotateSize": <RotateSize>,
         "rotateInterval": <RotateInterval>,   
         "format": <Format>,
         "delimiter": <Delimiter>,
         "compress": <true|false>,
         "retention": <Retention>
	}
}
```

**请求内容**

|参数|类型|必填|说明|
|:---|:---|:---:|:---|
|bucket|string|是|数据中心名称|
|keyPrefix|string|否|导出的文件名的前缀|
|email|string|是|数据中心名称所属用户的七牛账户名称|
|accessKey|string|是|七牛账户的公钥|
|fields|map|是|字段关系说明</br>`key`为`kodo-bucket`的字段名</br>`value`为导出数据的消息队列的字段名|
|rotateStrategy|string|否|文件切割策略，可取值为`size`、`interval`、`both`，其中，`size`表示文件大小超过`rotateSize`触发切割行为；`interval`表示文件写时长超过`rotateInterval`将进行切割；`both`表示只要满足其中一个条件将触发切割行为。为了保持兼容性，默认值为`interval`|
|rotateSize|int|否|当文件大小超过该值时将触发切割行为，单位为字节,默认值为`5242880`(5MB)，最大值不超过`1073741824`(1GB)|
|rotateInterval|int|否|文件切割间隔，单位为秒(`s`)，默认值为`600`(10分钟)|
|format|string|否|文件导出格式</br>支持`json`、`text`、`parquet`、`csv`四种形式</br>默认为`json`|
|delimiter|string|否|csv文件分割符，当文件类型为csv时，delimiter为必填项|
|compress|bool|否|是否开启文件压缩功能</br>默认为`false`|
| retention |int|否|数据储存时限</br>以天为单位</br>当不大于0或该字段为空时，则永久储存|

!> 注1: `compress` 会压缩成`gzip`格式,但当用户指定`format`为`parquet`时,由于`parquet`已经是压缩好的列存格式,`compress`选项将不起作用。

!> 注2: `keyPrefix`字段表示导出文件名称的前缀,该字段可选,默认值为""(生成文件名会自动加上时间戳格式为`yyyy-MM-dd-HH-mm-ss`),如果使用了一个或者多个魔法变量时不会自动添加时间戳,支持魔法变量,采用`$(var)`的形式求值,目前可用的魔法变量var如下:

* `year` 上传时的年份
* `mon` 上传时的月份
* `day` 上传时的日期
* `hour` 上传时的小时
* `min` 上传时的分钟
* `sec` 上传时的秒钟

>举例说明:
>
* 假如keyPrefix取值为kodo-parquet/date=$(year)-$(mon)-$(day)/hour=$(hour)/min=$(min)/$(sec),且生成某一文件时的北京标准时间为`2017-01-12 15:30:00`, 则keyPrefix将被解析为kodo-parquet/date=2017-01-12/hour=15/min=30/00,其中的魔法变量$(year)、$(mon)、$(day)、$(hour)、$(min)、$(sec)分别对应文件生成时间`2017-01-12 15:30:00`的年、月、日、时、分、秒。
* 假如keyPrefix使用默认值,且生成某一文件时的北京标准时间为`2017-01-12 15:30:00`, 则keyPrefix将被解析为`2017-01-12-15-30-00`。 

**示例**

```
curl -X POST https://nb-pipeline.qiniuapi.com/v2/repos/repox/exports/export1 \
-H 'Content-Type: application/json' \
-H 'Authorization: Pandora 2J1e7iG13J66GA8vWBzZdF-UR_d1MF-kacOdUUS4:NTi3wH_WlGxYOnXsvgUrO4XMD6Y='  \
-d '{
	"type": "kodo", 
	"spec": { 
		"bucket": "bucket2", 
		"keyPrefix": "key1/", 
		"email": "xiaoming@qiniu.com",
		"accessKey": "7H4FDc1M-FxXFZKwjbN_Up1OfY7DotXDjaM5jXzm",
		"fields": { 
			"f1": "#f1", 
			"f2": "#f2" 
		} 
		"format":"json",
		"compress": true
}' 
```

### 流式导出数据至报表平台

**请求语法**

```
POST /v2/repos/<RepoName>/exports/<ExportName>
Content-Type: application/json
Authorization: Pandora <auth>
{
  "type": <report>,
  "whence": <ExportWhence>,
  "spec": {
        "dbName": <DBName>,
        "tableName": <TableName>,
        "columns": {
            "column1": <#key1>,
            "column2": <#key2>,
            ...
        }
     }
}
```
**请求内容**

|参数|类型|必填|说明|
|:---|:---|:---:|:---|
| database |string|是|数据库名称|
| tableName |string|是|数据表名称|
| columns |map|是|字段关系说明</br> `keyN`表示源消息队列字段名称</br>`columnN`表示报表服务数据表字段名称|

### 更新流式导出任务

**请求语法**

```
PUT /v2/repos/<RepoName>/exports/<ExportName>
Content-Type: application/json
Authorization: Pandora <auth>
{
    "spec": <Spec>
}
```
!> **注意:仅支持对 `spec` 的修改（不允许修改`type`、`whence`）** 

### 查看所有流式导出任务

**请求语法**

```
GET /v2/repos/<RepoName>/exports
Authorization: Pandora <auth>
```

**响应报文** 

```
Content-Type: application/json
{
    "name": <ExportName>,
    "type": <ExportSchema>,
    "spec": <Spec>,
    "whence": <ExportWhence>
}
```

### 根据名称查看流式导出任务

**请求语法**

```
GET /v2/repos/<RepoName>/exports/<ExportName>
Authorization: Pandora <auth>
```

### 根据名称删除流式导出任务

**请求语法**

```
DELETE /v2/repos/<RepoName>/exports/<ExportName>
Authorization: Pandora <auth>
```

### 错误代码及相关说明

| 错误码 | 错误描述 |
| :---  | :----- |
|500	|服务器内部错误 |
|411	|E18003: 缺少内容长度|
|400	|E18004: 无效的内容长度|
|413	|E18005: 请求实体过大|
|400	|E18006: 请求实体为空|
|400	|E18007: 请求实体格式非法|
|400	|E18008: 字段长度超过限制|
|409	|E18101: 仓库已经存在|
|404	|E18102: 仓库不存在|
|400	|E18103: 无效的仓库名称|
|400	|E18104: 无效的日期格式|
|400	|E18105: 仓库Schema为空|
|400	|E18106: 无效的字段名称|
|400	|E18107: 不支持的字段类型|
|400	|E18108: 源仓库数量超过限制|
|400	|E18109: 无效的仓库模式|
|400	|E18110: 无效的字段格式|
|404	|E18111: 字段不存在|
|400	|E18112: 仓库上存在着级联的转换任务或者导出任务|
|409	|E18113: 仓库处于删除状态中|
|400	|E18117: Plugin名称不合法|
|404	|E18120: 共享资源池不存在|
|404	|E18122: 导出的仓库在logd中不存在|
|202	|E18124: 仓库处于创建中|
|400	|E18125: 读取gzip的打点请求体出错|
|409	|E18201: 计算任务已经存在|
|404	|E18202: 计算任务不存在|
|415	|E18203: 计算任务类型不支持|
|409	|E18204: 计算任务的源仓库与目的仓库相同|
|409	|E18205: 目的仓库已经被其他转换任务占用|
|400	|E18206: 目的仓库必须通过删除计算任务的方式删除|
|400	|E18207: 计算任务描述格式非法|
|400	|E18208: 计算任务interval非法|
|400	|E18209: 计算任务中的SQL语句非法|
|400	|E18211: 计算任务中plugin输出字段类型非法|
|400	|E18212: 仓库的区域信息和数据中心不相符|
|400	|E18213: 计算任务中容器类型非法|
|400	|E18214: 计算任务中容器数量非法|
|409	|E18215: 共享资源池处于使用中|
|404	|E18216: Plugin不存在|
|409	|E18217: Plugin已存在|
|409	|E18218: 共享资源池已存在|
|400	|E18219: Plugin上传内容长度不一致|
|400	|E18220: Plugin上传内容的MD5不一致|
|400	|E18221: 共享资源池名称非法|
|400	|E18222: 共享资源池的区域信息不一致|
|400	|E18223: 不能向计算任务的目标仓库中打点|
|409	|E18301: 导出任务已经存在|
|404	|E18302: 导出任务不存在|
|400	|E18303: 提交导出任务失败|
|400	|E18304: 删除导出任务失败|
|400	|E18305: 导出任务出现错误|
|401 |bad token：鉴权不通过 、token已过期、机器时间未同步|






