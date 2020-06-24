# Elasticsearch

## 		mappings 动态映射

### Dynamic fieid mappings 默认情况下是根据es支持的数据类型类推测参数的类型，而动态模板允许您定义自定义映射规则，根据自定义的规则来推测字段值所属的类型，从而添加字段类型映射。

```json
{
    "dynamic_templates": [	// 1
        {
            "my_template_name": {	//2
                match conditions ,	//3
                "mapping": {	    //4
                    "type": "keyword"
                }
            }
        }
    ]
}
```

1. 在类型映射时通过dynamic_templates属性定义动态映射模板，其类型为数据
2. 定义动态映射模板名称
3. 匹配条件，其定义方式包括：match_mapping_type，match，match_pattern，unmatch，path_match，path_unmatch
4. 匹配3的字段使用类型映射定义(映射参数为类型映射中支持的参数)

### 动态类型模板映射机制

1. match_mapping_type

   使用JSON解析器解析字段值的类型，由于JSON不能区分long和integer，也不允许区分double和float，所以它总是选择更广泛的数据类型，在使用字段动态映射时，es会将字段动态映射为long而不是integer类型，那如何将数字5动态映射为integer类型呢，利用match_mapping_type可以实现，如果将所有整数字段映射为整数而不是long，并将所有字符串字段映射为文本和关键字。。

   ```json
   PUT my_index
   {
       "mappings": {
           "_doc": {
               "dynamic_template": [
                   {
                       "integers": {
                           "match_mapping_type": "long",
                           "mapping": {
                               "type": "integer"
                           }
                       }
                   },
                   {
                       "strings": {
                           "match_mapping_type": "string",
                           "mapping": {
                               "type": "text",
                               "fields": {
                                   "raw": {
                                       "type": "keyword",
                                       "ignore_above": 256
                                   }
                               }
                           }
                       }
                   }
               ]
           }
       }
   }
   ```

   match_mapping_type为字段动态映射(字段类型检测)得出的类型建立一个映射关系，将该类型转换为mapping定义中的类型。

2. match and unmatch

   1. match 参数使用模式匹配字段名，而unmatch使用模式排除匹配的字段。

      ```json
      PUT my_index
      {
          "mappings": {
              "_doc": {
                  "dynamic_templates": [
                      {
                          "longs_as_strings": {
                              "match_mapping_type": "string",		 //1
                              "match": "*",					   //2
                              "unmatch": "*_text",			    //3
                              "mapping": {					   //4
                                  "type": "integer"
                              }
                          }
                      }
                  ]
              }
          }
      }
      PUT my_index/_doc/1
      {
        "long_num": "5",       //5
        "long_text": "foo"      //6
      }
      ```

      1. 表示该自动映射模板针对的字段为JSON解析器检测字段的类型为string的新增字段。
      2. 字段名称以long_开头的字段。
      3. 排除字段名称以_text的字段。
      4. 符合long_开头的字段，并不是以_text结尾的字段，如果JSON检索为string类型的新字段，映射为long。
      5. long_num，映射类型为long。
      6. long_text满足long_ 开头，但是以 _text结尾，故该字段不会映射为long，而是保留其JSON检测到的类型string，会映射为text字段和keyword多字段。

3. match_pattern

   1. 使用正则表达式来匹配字段名称。

      ```json
      {
          "dynamic_templates": [
              {
                  "longs_as_strings": {
                      "match_mapping_type": "string",
                      "match_pattern": "regex",	//1
                      "match":"^profit_\d+$",		//2
                      "mapping": {
                          "type": "long"
                      }
                  }
              }
          ]
      }
      ```

      1. 设置匹配模式为regex代表Java正则表达式
      2. Java正则表达式

4. patch_match and path_unmatch

   1. path_match 与 path_unmatch的工作方式与match，unmatch一样，只不过path_match是针对字段的全路径，特别是针对嵌套类型

      例子：将name下的字段除了middle字段为copy到name属性并列的full_name字段中。

      ```json
      PUT my_index
      {
          "mappings": {
              "_doc": {
                  "dynamic_templates": [
                      {
                          "copy_full_name": {
                              "path_match": "name.*",
                              "path_unmatch": "*.middle",
                              "mapping": {
                                  "type": "text",
                                  "copy_to": "full_name"
                              }
                          }
                      }
                  ]
              }
          }
      }
      PUT my_index/_doc/1
      {
          "name": {
              "first": "Alice",
              "middle": "Mary",
              "last": "White"
          }
      }
      ```

5. {name} and {dynamic_type}

   1. {name}展位符，表示字段的名称。

   2. {dynamic_type}：JSON解析器解析到字段类型。

      ```json
      PUT my_index
      {
          "mappings": {
              "_doc": {
                  "dynamic_templates": [
                      {
                          "named_analyzers": {		//1
                              "match_mapping_type": "string",
                              "match": "*",
                              "mapping": {
                                  "type": "text",
                                  "analyzer": "{name}"
                              }
                          }
                      },
                      {
                          "no_doc_values": {			//2
                              "match_mapping_type": "*",
                              "mapping": {
                                  "type": "{dynamic_type}",
                                  "doc_values": false
                              }
                          }
                      }
                  ]
              }
          }
      }
      PUT my_index/_doc/1
      {
        "english": "Some English text", 
        "count":   5 
      }
      ```

      1. 映射模板的含义为：对所有匹配到的字符串类型，类型映射为text，对应的分析器的名称与字段名相同，这个在使用时慎重，可能不存在同名的分词器。
      2. 对于匹配到的任何类型，其映射定义为类型为自动检测的类型，并且禁用doc_values=false

