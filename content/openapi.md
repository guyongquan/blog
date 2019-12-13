---
title: "OpenAPI"
date: 2019-12-11T14:54:44+08:00
draft: true
---
[OpenAPI](https://www.openapis.org/)是一个用于描述REST API的标准。  
### 举个例子
假如有如下创建用户的API：  
POST /users  
请求体示例：
```json
{"username":"newusername","password":"UserPassword@123"}  
```
password字段要求只能包含大写字母小写字母数字长度在6到16之间    
请求成功则返回201，响应示例：
```json
{"userId":"97c85479-3a56-4449-a737-853885128fbe"}  
```
用户名已存在或者密码不符合要求则返回400  
要求用户必须有权限，否则返回401  
要描述清楚一个REST API，需要说清楚哪些东西了？  

+ URI  
+ HTTP Method  
+ 参数信息
+ 请求体结构
+ 各种情况下的HTTP状态码和响应体结构  
……

OpenAPI定义了如何使用JSON/YAML精确的描述上述这些REST API信息。
```yaml
openapi: 3.0.2
info:
  title: example
  version: 0.0.1
paths:
  # URI
  /users:
    # HTTP Method
    post:
      requestBody:
        content:
          application/json:
            # 请求体结构
            schema:
              type: object
              properties:
                username:
                  type: string
                password:
                  type: string
                  pattern: "^[a-zA-Z0-9]{6,16}$"
            # 请求体体示例
            example:
              username: newusername
              password: UserPassword@123
      responses:
        # 各种情况下的response
        201:
          description: 添加用户成功
          content:
            application/json:
              schema:
                type: object
                properties:
                  userId:
                    type: string
              example:
                userId: 97c85479-3a56-4449-a737-853885128fbe
        400:
          description: 用户已存在或者密码不符合要求
        401:
          description: 当前用户没有权限
```
### 背景知识简介：[JSON schema](https://json-schema.org/)，[YAML](https://yaml.org/)
JSON schema 是用于描述JSON结构的标准。
复制一段[JSON schema官网上的例子](https://json-schema.org/learn/getting-started-step-by-step.html)。  
对于如下JSON文档：  
```json
{
  "productId": 1,
  "productName": "A green door",
  "price": 12.50,
  "tags": [ "home", "green" ]
}
```
它的schema是这样的：  
```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "http://example.com/product.schema.json",
  "title": "Product",
  "description": "A product from Acme's catalog",
  "type": "object",
  "properties": {
    "productId": {
      "description": "The unique identifier for a product",
      "type": "integer"
    },
    "productName": {
      "description": "Name of the product",
      "type": "string"
    },
    "price": {
      "description": "The price of the product",
      "type": "number",
      "exclusiveMinimum": 0
    },
    "tags": {
      "description": "Tags for the product",
      "type": "array",
      "items": {
        "type": "string"
      },
      "minItems": 1,
      "uniqueItems": true
    }
  },
  "required": [ "productId", "productName", "price" ]
}
```
可见，JSON schema也是一个JSON文档，里面描述了JSON对象有那些属性，属性类型等等信息。  
YAML是JSON的超集（参见[Relation to JSON](https://yaml.org/spec/1.2/spec.html#id2759572)）。YAML设计上更重视可读性，比如YAML支持注释，而JSON不支持。  
这意味着，很多情况下，能使用JSON的地方就能使用YAML。JSON schema也适用于YAML。  
VSCode对JSON schema提供了很好的支持（参见文档[JSON schemas and settings](https://code.visualstudio.com/docs/languages/json#_json-schemas-and-settings)）。  
在VSCode中，通过配置JSON schema，VSCode能在编辑JSON/YAML的时候根据配置的schema给出输入提示。

### OpenAPI与JSON schema
OpenAPI提供了[用于描述OpenAPI YAML/JSON文档的JSON schema](https://github.com/OAI/OpenAPI-Specification/tree/master/schemas)。  
所以，可以通过在VSCode中配置OpenAPI JSON schema从而在使用VSCode编写OpenAPI文档能获得输入提示。  
关于OpenAPI与JSON schema，有一点需要特别注意，OpenAPI标准中本身也有[schema对象](http://spec.openapis.org/oas/v3.0.2#schema-object)，用于描述请求体/响应体/参数结构。但是，OpenAPI中的schema，只是JSON Schema的子集。  
OpenAPI中的schema对象源自JSON schema，其属性都是来自JSON schema。但是有的地方做了调整。[schema对象](http://spec.openapis.org/oas/v3.0.2#schema-object)文档中说明了哪些做了调整

### OpenAPI与swagger
swagger系列工具，是OpenAPI的实现之一。
其中开源的swagger UI，swagger editor只支持OpenAPI 2。  
swagger hub支持OpenAPI 3，而且功能很强大，对API设计的整个流程支持都很好。  
但是……它是商业版的，国内访问很慢。  
[list of 3.0 implementations](https://github.com/OAI/OpenAPI-Specification/blob/master/IMPLEMENTATIONS.md)页面列出了一系列工具。  
这里选择使用VSCode编辑OpenAPI文档。  

### 配置VSCode
* 配置OpenAPI schema以获得输入提示  
  文件 => 首选项 => 设置 => 搜索 json:schema => 在 settings.json 中编辑  
  新增配置：
  ```json
  {
    "json.schemas": [
        {
            "fileMatch": [
                "*.oas3.json",
                "*.oas3.yaml",
                "*.oas3.yml"
            ],
            "url": "https://raw.githubusercontent.com/OAI/OpenAPI-Specification/master/schemas/v3.0/schema.json"
        }
    ]
  }
  ```
  经过上面的配置，当使用VSCode编辑扩展名为oas3.json/oas3.yaml/oas3.yml的文件时，VSCode就会根据OpenAPI中的schema给出输入提示。  
  ![VSCode输入提示](/images/openapi/vscode-typing.png)
* 安装插件[OpenAPI (Swagger) Editor](https://marketplace.visualstudio.com/items?itemName=42Crunch.vscode-openapi)  
  具体用法参照插件页面。  

### OpenAPI示例
#### Path参数示例
```yaml
  /pathparamexample/{path_param_1}/part1/{path_param_2}/part2:
    description: 路径参数示例
    parameters:
      - in: path
        # Path中的参数，required属性必须为true
        required: true
        name: path_param_1
        schema: 
          type: string
      - in: path
        required: true
        name: path_param_2
        schema: 
          type: integer
    get:
      responses:
        200:
          description: 操作成功
```

#### 对象复用示例
下面的例子中在components下定义了一个名为uuid的参数，和一个名为UserInfo的schema对象。
并在多处引用。
```yaml
openapi: 3.0.1
info:
  title: example
  version: 0.0.1
paths:
  /users:
    get:
      responses:
        200:
          description: 获取用户列表
          content:
            application/json:
              schema:
                type: array
                items: 
                  $ref: "#/components/schemas/UserInfo"
  /users/{uuid}:
    parameters:
      - $ref: "#/components/parameters/uuid"
    get:
      responses:
        200:
          description: 获取指定用户信息
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/UserInfo"
  /groups/{uuid}:
    parameters:
      - $ref: "#/components/parameters/uuid"
    get:
      responses:
        200:
          description: 用户组信息
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/UserInfo"
components:
  parameters:
    uuid:
      name: uuid
      in: path
      required: true
      schema:
        type: string
  schemas:
    UserInfo:
      type: object
      properties:
        name:
          type: string
        uuid:
          type: string
    GroupInfo:
      type: object
      properties:
        name: 
          type: string
        users:
          $ref: "#/components/schemas/UserInfo"
```

#### 子类继承示例
schema中定义data type的时候没有继承的概念，但是可以通过allOf加$ref让一个data type拥有另外一个或者多个data type的字段。  
比如下面的示例中，QQUser和WeiboUser都通过allOf引入了UserInfo的字段。
```yaml
components:
  schemas:
    UserInfo:
      type: object
      properties:
        name:
          type: string
        uuid:
          type: string
    QQUser: 
      allOf:
        - $ref: "#/components/schemas/UserInfo"
        - type: object
          properties:
            qq: 
              type: string
              pattern: "^\\d+$"
    WeiboUser:
      allOf:
        - $ref: "#/components/schemas/UserInfo"
        - type: object
          properties:
            weibo: 
              type: string
```
#### Query中的json格式参数示例
当在Query String中定义json格式的参数时，需要在参数的content中定义，而不是直接在schema中定义。
```yaml
openapi: 3.0.1
info:
  title: example
  version: 0.0.1
paths:
  /users:
    get:
      parameters:
        - name: common_param
          in: query
          # 普通的参数，直接在schema字段里定义类型。
          schema:
            type: string
        - name: filter
          in: query
          # json格式的参数，在content中定义。
          content:
            application/json:
              schema:
                type: object
                properties:
                  field:
                    type: string
                  value: 
                    oneOf:
                      - type: string
                      - type: number
              examples:
                number:
                  description: 数值类型的字段过滤设置
                  value:
                    field: stars
                    value: 100
                string:
                  description: 字符串类型的字段过滤设置
                  value:
                    field: country
                    value: China
      responses:
        200:
          description: 获取用户列表
          content:
            application/json:
              schema:
                type: array
                items: 
                      type: object
                      properties:
                        name:
                          type: string
                        uuid:
                          type: string
```