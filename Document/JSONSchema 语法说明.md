JSON模式是基于JSON格式定义JSON数据结构的规范。它被写在IETF草案，于2011年到期。 JSON模式：

JSON Schema 定义了如何基于 JSON 格式描述 JSON 数据结构的规范，进而提供数据校验、文档生成和接口数据交互控制等一系列能力。它的特性和用途，可以大致归纳为以下几点：

1. 用于描述数据结构

    在描述 JSON 数据时，如果数据本身的复杂度很高，高到三维四维，普通的标签函数已经无法表示这种层级结构了，而 JSON Schema 利用 object 和 array 字段类型的反复嵌套，可以规避掉这个缺陷。
    当然，除了键值等基本信息，规范层面还提供了丰富的关键词支持，如果想通过自定义扩展字段，解决特定场景的业务需求，也是非常方便的。

2. 用于构建人机可读的文档

    计算机领域有个概念叫做自描述。所谓自描述，可以理解为：文档本身包含了自身与其他文档交互相关的描述信息，不需要其他的配置文件或者额外信息来描述。
    而 JSON Schema 就是自描述的，它本身就是一份很完善的说明文档，字段的含义说明、该如何取值、格式的要求等都清晰明了。

3. 用于生成模拟数据

    通过标签函数生成模拟数据，只能解决基本的格式要求。比如 string 类型的字段，模拟出来的数据，无非是一个随机字符串。
    但在 JSON Schema 中，由于字段的描述不仅仅是类型，更多的约束条件，可以确保模拟数据更接近于真实数据。

4. 用于校验数据，实现自动化测试

    接口数据的校验工作，往往依赖于测试代码逻辑和用例。如果用 JSON Schema 描述一个数据接口，就不需要再编写测试代码了，所有的逻辑都可以移植到 JSON Schema 中维护。配合 jsv、tv4 等二方校验工具，接口测试可以真正自动化。



关于JSON Schema的语法，可以查看：http://json-schema.org/

对于基础学习，可以查看：https://spacetelescope.github.io/understanding-json-schema/index.html

但是，我想说的是，上面的都有点复杂， 所以我想写一个比较全的例子，来方便理解和使用：
```json
{
    "$schema": "http://json-schema.org/schema#",  //$schema 关键字，表明该文档是一个JSONSchema文档，与一般的JSON文档 作为区别
    "$id": "demo.json",                           //文档的ID， 方便引用外部文档的内容，使用 "$ref" : "demo.json#/definitions/Customer"

    "definitions": {                              //JSON Schema中的对象定义
        "Customer": {
            "type": "object",                     //对象类型，基本类型有： object， string，number，integer,boolean, array, null
            "title": "Customer",                  //对象名称
            "description": "this is a Customer definition",
            "properties": {
                "name": {
                    "type": "string",
                    "title": "Name",
                    "default": "Default value",
                    "description": " customer name",
                    "enum": ["red", "amber", "green", null],
                    "pattern": "^(\\([0-9]{3}\\))?[0-9]{3}-[0-9]{4}$",
                    "minLength": 2,
                    "maxLength": 3,
                    "format": "date-time|email|hostname|ipv4|ipv6|uri"
                },
                "age": {
                    "type": "number", //integer

                    "title": "Age",
                    "default": 1,
                    "description": " customer age",
                    "enum": [1, 2, 3, null],

                    "minimum": 0,
                    "maximum": 100
                },

                "isMale": {
                    "type": "boolean",
                    "title": "Age",
                    "default": true,
                    "description": " customer gender"
                },

                "id": {
                    "type": "integer",
                    "readOnly": true
                },

                "addresses": {
                    "type": "array",

                    "title": "Name",
                    "default": "Default value",
                    "description": " customer addresses",

                    "minItems": 2,
                    "maxItems": 3,                                   //关键字maxItems，minItems，用于校验数组长度最大，最小值
                    "uniqueItems": false,                            //关键字uniqueItem，判断数组中对象是否唯一
                    "items": {
                        "$ref": "#/definitions/Address" ,             //关键字”$ref“，用于支持对象引用能力
                         "additionalItems": false                     //关键字”additionalItems“，对于有限类型的数组，是否支持其他类型的值
                    }
                },

                "additionalProperties": true                         //关键字”additionalProperties“，对象是否支持用户自定义扩展字段
            },
            "required": ["name"]                                     //关键字required， 指定对象中的必填字段
        },
        "Address": {
            "type": "object",
            "title": "Customer",
            "description": "this is a Customer definition",
            "properties": {
                "street_address": { "type": "string" },
                "city": { "type": "string" },
                "state": { "type": "string" }
            },
            "required": ["street_address", "city", "state"]
        },

        "shipping_address": {
            "allOf": [                                               //关键字allof，用于支持对象继承
                // Here, we include our "core" address schema...
                { "$ref": "#/definitions/Address" },

                // ...and then extend it with stuff specific to a shipping
                // address
                {
                    "properties": {
                        "type": {
                            "enum": ["residential", "business"]
                        }
                    },
                    "required": ["type"]
                }
            ]
        }
    }
}

```

总结一下：

JSON Schema 中

- 基本数据类型有：

        string
        Numeric types
        object
        array
        boolean
        null
 
 - 每个数据类型都支持的关键字有：
 
    1.  描述信息相关的关键字：
    ```json
    {
        "title" : "Match anything",
        "description" : "This is a schema that matches anything.",
        "default" : "Default value"
    }
    ```
    2. 枚举值相关关键字：
    ```json
    {
        "enum": ["red", "amber", "green"]
    }
    ```
    
  - 每个数据类型各自支持的关键字有：
  
    1. string类型支持的特有关键字
        ```json
        { "type": "string", 
		  "pattern": "^(\\([0-9]{3}\\))?[0-9]{3}-[0-9]{4}$", 
		  "minLength": 2, 
		  "maxLength": 3, 
		  "format": "date-time|email|hostname|ipv4|ipv6|uri" 
        }
        ```
     2. number类型支持的特有关键字
        ```json
        {
            "type": "number",
            "minimum": 0,
            "maximum": 100,
            "multipleOf" : 10,
            "exclusiveMaximum": true
        }
        ```
     3. object 类型支持的特有关键字
        ```json
        { 
            "type": "object",
            "properties": {
                "name": { "type": "string" },
			    "credit_card": { "type": "number" }
		    }
			"required": ["name"] 
        }
        ```
     4. array 类型支持的特有关键字
        ```json
        { 
            "type": "array",
            "minItems": 2,
            "maxItems": 3, 
            "items": 
            {
                "type": "number"
			}
        }
        ```
        
### 关键字集合 ###
 ```json
{
    "$schema": "http://json-schema.org/schema#",
    "$comment": "",
    "$ref": "",
    "$id": "demo.json",   
    "definitions":{
    }
    "additionalItems": true,
    "additionalProperties": true,
    "allOf": [
        
    ],
    "anyOf": [
        
    ],
    "const":"",
    "contains": true,
    "contentEncoding": "",
    "contentMediaType": "",
    "default":"",
    "dependencies": {
        
    },
    "description": "",
    "else": true,
    "enum": [
        
    ],
    "examples": [
        
    ],
    "exclusiveMaximum": 0,
    "exclusiveMinimum": 0,
    "format": "",
    "if": true,
    "items": true,
    "maximum": 0,
    "maxItems": 0,
    "maxLength": 0,
    "maxProperties": 0,
    "minimum": 0,
    "minItems":1,
    "minLength":0,
    "minProperties":0,
    "multipleOf": 0,
    "not": true,
    "oneOf": [
        
    ],
    "pattern": "",
    "patternProperties": {
        
    },
    "properties": {
        
    },
    "propertyNames": true,
    "readOnly": false,
    "required": [
        
    ],
    "then": true,
    "title": "",
    "type":"array",
    "uniqueItems": false
}
```
