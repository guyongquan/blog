---
title: "OpenAPI"
date: 2019-12-11T14:54:44+08:00
draft: true
---
[OpenAPI](https://www.openapis.org/)是一个用于描述REST API的标准。  
### 举个例子
假如有如下创建用户的API：  
POST /users  
请求体示例：{"username":"newusername","password":"UserPassword@123"}  
password字段要求只能包含大写字母小写字母数字长度在6到16之间    
请求成功则返回201，响应示例：{"userId":"97c85479-3a56-4449-a737-853885128fbe"}  
用户名已存在或者密码不符合要求则返回400 
要求用户必须有权限，否则返回401 
上面是用自然语言描述一个API。  
如果是用OpenAPI：  
```yaml
openapi: 3.0.2
info:
  title: example
  version: 0.0.1
paths:
  /users:
    post:
      requestBody:
        content:
          application/json:
            schema:
              type: object
              properties:
                username:
                  type: string
                password:
                  type: string
                  pattern: "^[a-zA-Z0-9]{6,16}$"
            example:
              username: newusername
              password: UserPassword@123
      responses:
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
### 背景知识简介：[JSON schema](https://json-schema.org/)，[yaml](https://yaml.org/)
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
yaml是JSON的超集（参见[Relation to JSON](https://yaml.org/spec/1.2/spec.html#id2759572)）。yaml设计上更重视可读性，比如yaml支持注释，而JSON不支持。  
这意味着，很多情况下，能使用JSON的地方就能使用yaml。JSON schema也适用于yaml。  
VSCode对JSON schema提供了很好的支持（参见文档[JSON schemas and settings](https://code.visualstudio.com/docs/languages/json#_json-schemas-and-settings)）。  
在VSCode中，通过配置JSON schema，VSCode能在编辑JSON/yaml的时候根据配置的schema给出输入提示。
### OpenAPI与JSON schema
从上面的例子可以看出，OpenAPI文档是一个yaml/JSON格式的。OpenAPI提供了[用于描述OpenAPI yaml/JSON文档的JSON schema](https://github.com/OAI/OpenAPI-Specification/tree/master/schemas)。  
所以，可以通过在VSCode中配置OpenAPI JSON schema从而在使用VSCode编写OpenAPI文档能获得输入提示。  
关于OpenAPI与JSON schema，有一点需要特别注意，OpenAPI标准中本身也有[schema对象](http://spec.openapis.org/oas/v3.0.2#schema-object)，用于描述请求体/响应体/参数结构。但是，OpenAPI中的schema，只是JSON Schema的子集。

### OpenAPI与swagger
swagger系列工具，是OpenAPI的实现之一。
其中开源的swagger UI，swagger editor只支持OpenAPI 2。  
swagger hub支持OpenAPI 3，而且功能很强大，对API设计的整个流程支持都很好。  
但是……它是商业版的，国内访问很慢。  
[list of 3.0 implementations](https://github.com/OAI/OpenAPI-Specification/blob/master/IMPLEMENTATIONS.md)页面列出了一系列工具。  
这里选择使用VSCode编辑OpenAPI文档。  

### 配置VSCode
* 配置schema  
  VSCode对yaml/JSON的支持很好，能通过配置JSON schema来实现输入提示。  
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

### Path参数示例
```yaml
  /pathparamexample/{path_param_1}/part1/{path_param_2}/part2:
    description: 路径参数示例
    parameters:
      - in: path
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

### 对象复用示例
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
  /users/{name}:
    parameters:
      - $ref: "#/components/parameters/name"
    get:
      responses:
        200:
          description: 获取指定用户信息
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/UserInfo"
  /groups/{name}:
    parameters:
      - $ref: "#/components/parameters/name"
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
    name:
      name: name
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

### 子类继承示例
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
### Query中的json格式参数示例
```yaml
openapi: 3.0.1
info:
  title: example
  version: 0.0.1
paths:
  /users:
    get:
      parameters:
        - name: filter
          in: query
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