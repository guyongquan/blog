---
title: "OpenAPI"
date: 2019-12-11T14:54:44+08:00
draft: true
---
# OpenAPI
[OpenAPI](https://www.openapis.org/)是一个用于描述REST API的标准。  
## OpenAPI简介
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
  
以上，是用自然语言描述一个REST API。其中包含了下面这些信息：

+ URI  
+ HTTP Method  
+ 参数信息
+ 请求体结构
+ 各种情况下的HTTP状态码和响应体结构  
……

OpenAPI定义了如何使用JSON精确的描述上述这些REST API信息。
```json
{
  "openapi": "3.0.2",
  "info": {
    "title": "example",
    "version": "0.0.1"
  },
  "paths": {
    "/users": {
      "post": {
        "requestBody": {
          "content": {
            "application/json": {
              "schema": {
                "type": "object",
                "properties": {
                  "username": {
                    "type": "string"
                  },
                  "password": {
                    "type": "string",
                    "pattern": "^[a-zA-Z0-9]{6,16}$"
                  }
                }
              },
              "example": {
                "username": "newusername",
                "password": "UserPassword@123"
              }
            }
          }
        },
        "responses": {
          "201": {
            "description": "添加用户成功",
            "content": {
              "application/json": {
                "schema": {
                  "type": "object",
                  "properties": {
                    "userId": {
                      "type": "string"
                    }
                  }
                },
                "example": {
                  "userId": "97c85479-3a56-4449-a737-853885128fbe"
                }
              }
            }
          },
          "400": {
            "description": "用户已存在或者密码不符合要求"
          },
          "401": {
            "description": "当前用户没有权限"
          }
        }
      }
    }
  }
}
```
## 背景知识简介：[JSON schema](https://json-schema.org/)
JSON schema 是一个规范，用于描述JSON对象结构：一个JSON对象有哪些字段，分别是什么类型，有什么约束最大值最小值……  
复制一段[JSON schema官网上的例子](https://json-schema.org/learn/getting-started-step-by-step.html)。  
比如下面这个JSON对象：  
```json
{
  "productId": 1,
  "productName": "A green door",
  "price": 12.50,
  "tags": [ "home", "green" ]
}
```
它有四个字段：

1. productId：产品id，整型  
2. productName： 产品名称，字符串  
3. price：价格，浮点型  
4. tags：标签，字符串数组，长度至少为1，元素不能重复  

JSON schema就是用于表达上述信息的：
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
VSCode对JSON schema提供了很好的支持（参见文档[JSON schemas and settings](https://code.visualstudio.com/docs/languages/json#_json-schemas-and-settings)）。  
在VSCode中，通过配置JSON schema，VSCode能在编辑JSON的时候根据配置的schema给出输入提示。

## OpenAPI与JSON schema
OpenAPI提供了[用于描述OpenAPI YAML/JSON文档的JSON schema](https://github.com/OAI/OpenAPI-Specification/tree/master/schemas)。  
所以，可以通过在VSCode中配置OpenAPI JSON schema从而在使用VSCode编写OpenAPI文档能获得输入提示。  
关于OpenAPI与JSON schema，有一点需要特别注意，OpenAPI标准中本身也有[schema对象](http://spec.openapis.org/oas/v3.0.2#schema-object)，用于描述请求体/响应体/参数结构。但是，OpenAPI中的schema，只是JSON Schema的子集。  
OpenAPI中的schema对象源自JSON schema，其属性都是来自JSON schema。但是有的地方做了调整。[schema对象](http://spec.openapis.org/oas/v3.0.2#schema-object)文档中说明了哪些做了调整

## OpenAPI与swagger
swagger系列工具是OpenAPI的实现之一。
其中开源的swagger UI，swagger editor只支持OpenAPI 2。  
商业版的swagger hub支持OpenAPI 3，而且功能很强大，对API设计的整个流程支持都很好。  
只是，国内访问很慢。  
[list of 3.0 implementations](https://github.com/OAI/OpenAPI-Specification/blob/master/IMPLEMENTATIONS.md)页面列出了一系列工具。  
这里选择使用VSCode编辑OpenAPI文档。  

## 配置VSCode
### 配置OpenAPI schema以获得输入提示  
文件 => 首选项 => 设置 => 搜索 json:schema => 在 settings.json 中编辑  
新增配置：
```json
{
  "json.schemas": [
      {
          "fileMatch": ["*.oas3.json"],
          "url": "https://raw.githubusercontent.com/OAI/OpenAPI-Specification/master/schemas/v3.0/schema.json"
      },
      {
          "fileMatch": ["*.oas3schema.json"],
          "url": "https://raw.githubusercontent.com/OAI/OpenAPI-Specification/master/schemas/v3.0/schema.json#/definitions/Components/properties/schemas"
      }
  ]
}
```
经过上面的配置，当使用VSCode编辑扩展名为oas3.json的文件时，VSCode就会根据schema.json中的定义给出输入提示，并检查JSON对象是否符合schema.json中的定义。  
  ![VSCode输入提示](/images/openapi/vscode-typing.png)
而对于*.oas3schema.json的文件，则会使用schema.json中/definitions/Components/properties/schemas定义的。  
这是为了方便编写对象定义文件，后面会提到。
上面的配置中url指向的都是github上的内容，因为国内访问github不稳定，所以也可以把文档下载下来配置本地路径。  
本地路径可以是相对于VSCode工作空间的相对路径，详情参考VSCode文档[Mapping to a schema in the workspace](https://code.visualstudio.com/docs/languages/json#_mapping-to-a-schema-in-the-workspace)。
### 安装插件[OpenAPI (Swagger) Editor](https://marketplace.visualstudio.com/items?itemName=42Crunch.vscode-openapi)  
  具体用法参照插件页面。  

## OpenAPI示例
### Path参数示例
```json
{
  "/pathparamexample/{path_param_1}/part1/{path_param_2}/part2": {
    "description": "路径参数示例",
    "parameters": [
      {
        "in": "path",
        "required": true,
        "name": "path_param_1",
        "schema": {
          "type": "string"
        }
      },
      {
        "in": "path",
        "required": true,
        "name": "path_param_2",
        "schema": {
          "type": "integer"
        }
      }
    ],
    "get": {
      "responses": {
        "200": {
          "description": "操作成功"
        }
      }
    }
  }
}
```

### 对象复用
#### 对象复用概述
OpenAPI中可以在components节点下定义可复用对象。  
从components节点下弹出的输入提示我们可以看到OpenAPI3中的可复用对象类型：
![components下的节点](/images/openapi/components.png)
#### 对象复用示例
下面的例子中在components下定义了一个名为uuid的参数，和一个名为UserInfo的schema对象。
并在多处引用。
```json
{
  "openapi": "3.0.1",
  "info": {
    "title": "example",
    "version": "0.0.1"
  },
  "paths": {
    "/users": {
      "get": {
        "responses": {
          "200": {
            "description": "获取用户列表",
            "content": {
              "application/json": {
                "schema": {
                  "type": "array",
                  "items": {
                    "$ref": "#/components/schemas/UserInfo"
                  }
                }
              }
            }
          }
        }
      }
    },
    "/users/{uuid}": {
      "parameters": [
        {
          "$ref": "#/components/parameters/uuid"
        }
      ],
      "get": {
        "responses": {
          "200": {
            "description": "获取指定用户信息",
            "content": {
              "application/json": {
                "schema": {
                  "$ref": "#/components/schemas/UserInfo"
                }
              }
            }
          }
        }
      }
    },
    "/groups/{uuid}": {
      "parameters": [
        {
          "$ref": "#/components/parameters/uuid"
        }
      ],
      "get": {
        "responses": {
          "200": {
            "description": "用户组信息",
            "content": {
              "application/json": {
                "schema": {
                  "$ref": "#/components/schemas/UserInfo"
                }
              }
            }
          }
        }
      }
    }
  },
  "components": {
    "parameters": {
      "uuid": {
        "name": "uuid",
        "in": "path",
        "required": true,
        "schema": {
          "type": "string"
        }
      }
    },
    "schemas": {
      "UserInfo": {
        "type": "object",
        "properties": {
          "name": {
            "type": "string"
          },
          "uuid": {
            "type": "string"
          }
        }
      },
      "GroupInfo": {
        "type": "object",
        "properties": {
          "name": {
            "type": "string"
          },
          "users": {
            "$ref": "#/components/schemas/UserInfo"
          }
        }
      }
    }
  }
}
```

### 子类继承示例
schema中定义data type的时候没有继承的概念，但是可以通过allOf加$ref让一个data type拥有另外一个或者多个data type的字段。  
比如下面的示例中，QQUser和WeiboUser都通过allOf引入了UserInfo的字段。
```json
{
  "components": {
    "schemas": {
      "UserInfo": {
        "type": "object",
        "properties": {
          "name": {
            "type": "string"
          },
          "uuid": {
            "type": "string"
          }
        }
      },
      "QQUser": {
        "allOf": [
          {
            "$ref": "#/components/schemas/UserInfo"
          },
          {
            "type": "object",
            "properties": {
              "qq": {
                "type": "string",
                "pattern": "^\\d+$"
              }
            }
          }
        ]
      },
      "WeiboUser": {
        "allOf": [
          {
            "$ref": "#/components/schemas/UserInfo"
          },
          {
            "type": "object",
            "properties": {
              "weibo": {
                "type": "string"
              }
            }
          }
        ]
      }
    }
  }
}
```
### Query中的json格式参数示例
当在Query String中定义json格式的参数时，需要在参数的content中定义，而不是直接在schema中定义。
```json
{
  "openapi": "3.0.1",
  "info": {
    "title": "example",
    "version": "0.0.1"
  },
  "paths": {
    "/users": {
      "get": {
        "parameters": [
          {
            "name": "common_param",
            "in": "query",
            "schema": {
              "type": "string"
            }
          },
          {
            "name": "filter",
            "in": "query",
            "content": {
              "application/json": {
                "schema": {
                  "type": "object",
                  "properties": {
                    "field": {
                      "type": "string"
                    },
                    "value": {
                      "oneOf": [
                        {
                          "type": "string"
                        },
                        {
                          "type": "number"
                        }
                      ]
                    }
                  }
                },
                "examples": {
                  "number": {
                    "description": "数值类型的字段过滤设置",
                    "value": {
                      "field": "stars",
                      "value": 100
                    }
                  },
                  "string": {
                    "description": "字符串类型的字段过滤设置",
                    "value": {
                      "field": "country",
                      "value": "China"
                    }
                  }
                }
              }
            }
          }
        ],
        "responses": {
          "200": {
            "description": "获取用户列表",
            "content": {
              "application/json": {
                "schema": {
                  "type": "array",
                  "items": {
                    "type": "object",
                    "properties": {
                      "name": {
                        "type": "string"
                      },
                      "uuid": {
                        "type": "string"
                      }
                    }
                  }
                }
              }
            }
          }
        }
      }
    }
  }
}
```
### 数组类型示例
通过type: array定义数组类型，在items中定义数组元素的类型。
```json
{
  "openapi": "3.0.1",
  "info": {
    "title": "example",
    "version": "0.0.1"
  },
  "paths": {
    "/papers": {
      "get": {
        "responses": {
          "200": {
            "description": "ArrayExample",
            "content": {
              "application/json": {
                "schema": {
                  "type": "array",
                  "items": {
                    "type": "object",
                    "properties": {
                      "title": {
                        "type": "string"
                      },
                      "authorNames": {
                        "type": "array",
                        "items": {
                          "type": "string"
                        }
                      }
                    }
                  }
                },
                "examples": {
                  "example1": {
                    "value": [
                      {
                        "title": "title1",
                        "authorNames": [
                          "author1",
                          "author2"
                        ]
                      },
                      {
                        "title": "title2",
                        "authorNames": [
                          "author1",
                          "author3"
                        ]
                      }
                    ]
                  }
                }
              }
            }
          }
        }
      }
    }
  }
}

```
### 枚举类型
```json
{
  "openapi": "3.0.1",
  "info": {
    "title": "example",
    "version": "0.0.1"
  },
  "paths": {
    "/day": {
      "get": {
        "responses": {
          "200": {
            "description": "what day is it today",
            "content": {
              "text/plain": {
                "schema": {
                  "type": "string",
                  "enum": [
                    "MONDAY",
                    "TUESDAY",
                    "WEDNESDAY",
                    "THURSDAY",
                    "SATURDAY",
                    "SUNDAY"
                  ]
                }
              }
            }
          }
        }
      }
    }
  }
}
```
### 使用oneOf处理“type”字段
在一些情况下，JSON对象会因为某个字段的值不同而有不同的属性。此时可以用oneOf来处理。  
比如有如下后端Java DTO对象定义，并且有REST API GET /responsewithtypefield的返回值可能是Type1或者Type2：
``` java
enum SomeType {TYPE1, TYPE2}
class Base {
  public SomeType type;
  public int baseField;
  protected Base(SomeType pramType) {
    type = paramType;
  }
}

class Type1 extends Base {
  public boolean subclassField;
  public String type1Field;
  public Type1() {
    super(SomeType.TYPE1);
  }
}

class Type2 extneds Base {
  public int subclassField;
  public String type2Field;
  public Type2() {
    super(SomeType.TYPE2)
  }
}
```
则对应的Open API文档可以这么写：
```json
{
  "openapi": "3.0.1",
  "info": {
    "title": "example",
    "version": "0.0.1"
  },
  "paths": {
    "/responsewithtypefield": {
      "get": {
        "responses": {
          "200": {
            "description": "返回值类型可能是Type1或者Type2",
            "content": {
              "application/json": {
                "schema": {
                  "oneOf": [
                    {
                      "$ref": "#/components/schemas/Type1"
                    },
                    {
                      "$ref": "#/components/schemas/Type2"
                    }
                  ]
                },
                "examples": {
                  "type1": {
                    "value": {
                      "type": "TYPE1",
                      "subclassField": false,
                      "type1Field": "type1FieldValue"
                    }
                  },
                  "type2": {
                    "value": {
                      "type": "TYPE2",
                      "subclassField": 1,
                      "type2Field": "type2FieldValue"
                    }
                  }
                }
              }
            }
          }
        }
      }
    }
  },
  "components": {
    "schemas": {
      "Base": {
        "type": "object",
        "properties": {
          "type": {
            "type": "string",
            "enum": [
              "TYPE1",
              "TYPE2"
            ]
          }
        }
      },
      "Type1": {
        "allOf": [
          {
            "$ref": "#/components/schemas/Base"
          },
          {
            "type": "object",
            "properties": {
              "type": {
                "type": "string",
                "enum": [
                  "TYPE1"
                ]
              },
              "subclassField": {
                "type": "boolean"
              },
              "type1Field": {
                "type": "string"
              }
            }
          }
        ]
      },
      "Type2": {
        "allOf": [
          {
            "$ref": "#/components/schemas/Base"
          },
          {
            "type": "object",
            "properties": {
              "type": {
                "type": "string",
                "enum": [
                  "TYPE2"
                ]
              },
              "subclassField": {
                "type": "integer"
              },
              "type2Field": {
                "type": "string"
              }
            }
          }
        ]
      }
    }
  }
}
```
上述Type1和Type2的schema定义中，都使用了只有一个值的enum来实现常量。
这是因为当前版本（3.0.2）的OpenAPI schema object中没有const，后续[3.1版本会引入const value](https://github.com/OAI/OpenAPI-Specification/issues/1313#issuecomment-335898974)。
### 跨文件引用schema定义
在实际的产品开发中，会有不同的API需要放到不同的文件里，但是这些API又有相同的data type定义。  
对于这样的情况，比较合适的做法是把data type定义放到单独的文件中，然后通过[$ref](http://spec.openapis.org/oas/v3.0.2#reference-object)引用。  
在前面的示例中已经出现过#/components/schemas/xxxx这样的ref，这样的ref都是相对引用，引用的是当前文档中的对象。  
ref也可以引用其它文件中的对象。下面是一个简单的示例：  
三个文件：

* datatypes.oas3schema.json，包含datatype定义，其中定义了一个名为CommonType1的类型。
* moduleA.oas3.json，moduleA API定义。
* moduleB.oas3.json，moduleB API定义。

moduleA和ModuleB中都引用了datatypes.oas3schema.json#/CommonType1。

datatypes.oas3schema.json：  
只用于定义data type，没有写任何的API定义，它的内容结构跟其它两个文件不一样。  
```json
{
    "CommonType1": {
        "type": "object",
        "properties": {
            "p1": {
                "type": "string"
            },
            "p2": {
                "type": "number"
            }
        }
    }
}
```
moduleA.oas3.json：
```json
{
    "openapi": "3.0.2",
    "info": {
        "title": "Module A",
        "version": "0.0.1"
    },
    "paths": {
        "/moduleA": {
            "get": {
                "responses": {
                    "200": {
                        "description": "module A 200 response",
                        "content": {
                            "application/json": {
                                "schema": {
                                    "$ref": "datatypes.oas3schema.json#/CommonType1"
                                }
                            }
                        }
                    }
                }
            }
        }
    }
}
```
moduleB.oas3.json：
```json
{
    "openapi": "3.0.2",
    "info": {
        "title": "Module B",
        "version": "0.0.1"
    },
    "paths": {
        "/moduleB": {
            "post": {
                "requestBody":{
                    "content": {
                        "application/json": {
                            "schema": {
                                "$ref": "datatypes.oas3schema.json#/CommonType1"
                            }
                        }

                    }
                },
                "responses": {
                    "201": {
                        "description": "module B 201 response"
                    }
                }
            }
        }
    }
}
```

### 再谈allOf,oneOf
[allOf](https://tools.ietf.org/html/draft-wright-json-schema-validation-00#section-5.22)和[oneOf](https://tools.ietf.org/html/draft-wright-json-schema-validation-00#section-5.24)都是是JSON schema规范中定义的关键字，都是一个数组，数组中的元素都是JSON schema对象。allOf的作用是要求JSON对象符合这个数组中所有JSON schema的定义，oneOf要求JSON对象符合数组中其中一个并且只能符合其中一个JSON schema的定义。  
举个关于allOf的例子，如果要求密码符合下列要求：

* 类型是字符串
* 只能包含大写字母小写字母和数字
* 必须包含大写字母小写字母和数字
* 不能是完全相同的字符比如aaaaa

要定义一个Password，除了写一个超级复杂的正则表达式之外，还可以这样： 
```json
{
    "openapi": "3.0.2",
    "info": {
        "title": "allOf example",
        "version": "0.0.1"
    },
    "paths": {
        "/password": {
            "put": {
                "description": "update password",
                "requestBody":{
                    "content": {
                        "text/plain": {
                            "schema": {
                                "$ref": "#/components/schemas/Password"
                            }
                        }
                    }
                },
                "responses": {
                    "204": {
                        "description": "success"
                    }
                }
            }
        }
    },
    "components": {
        "schemas": {
            "Password": {
                "allOf": [
                    {
                        "type": "string"
                    },
                    {
                        "description": "只能包含大小写字母和数字",
                        "pattern": "^[A-Za-z0-9]*$"
                    },
                    {
                        "description": "必须包含大写字母",
                        "pattern": "^.*[A-Z]+.*$"
                    },
                    {
                        "description": "必须包含小写字母",
                        "pattern": "^.*[a-z]+.*$"
                    },
                    {
                        "description": "必须包含数字",
                        "pattern": "^.*[0-9]+.*$"
                    },
                    {
                        "description": "不能是完全重复的字符",
                        "not": {
                            "pattern": "^(.)\\1+$"
                        }
                    }
                ]
            }
        }
    }
}
```
上述allOf数组有6个JSON schema，每一个都是一个JSON schema对象：1个type，4个pattern，1个取反的pattern。allOf要求JSON对象符合全部6个schema。  
oneOf则要求符合其中一个并且只能是一个。anyOf则是至少一个。  
JSON schema对象和JSON对象的基本关系有两种可能：

1. JSON对象符合JSON schema
2. JSON对象不符合JSON schema

而通过allOf，oneAll，anyOf，not，可以组合出复杂的JSON schema对象，应对复杂的JSON对象结构。  
  


**特别注意，文中涉及到JSON schema的示例，都是以OpenAPI schema对象为基准。而不是JSON schema标准。虽然因为OpenAPI schema是JSON schema的子集，所以里面的示例也符合JSON schema标准。但是JSON schema标准不只有这些。**