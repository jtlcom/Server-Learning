# 编码规范

## 目录

* [JSON](#json)
* [JavaScript](#javascript)
* [Elixir](#elixir)

## JSON

```json
//s世界等级.json
[
  { //两格缩进
    "jsonFileName": "world_level.json", //采用单词全小写加下划线(unix_like)命名
    "docName": "world_level:id",    //名字与上面一致
    "shouldObject": false,
    "sheet": "s世界等级",   //与csv配置表的名字对应，否则无法转表
    "mapping": {
      "id": "天数", //前面的名字自取，采用单词全小写加下划线(unix_like)命名
      "world_level": "世界等级" //后面的与csv配置表中的属性(每一列的名字)一一对应，尽量保证顺序对应
    },
    "type": {
      "id": "int",  //与mapping的命名一致
      "world_level": "int"  //类型可用int, float, string
    }
  }
]   //mapping中的id会成为CouchDB中的key值，所以尽量选择csv中的索引(或者易于区分且唯一的值)作为JSON的id
```

## JavaScript

```javascript
function (doc) {  //两格缩进
  var [type, id] = doc._id.split(':')
  if (type === 'world_level') {   //与JSON的docName对应
    var config = doc.value
    //客户端需要将键的首字母大写，而服务器不需要
    var {uppercaseFirst} = require('views/lib/common')
    //其他常用函数，可在CouchDB的离散数据表中查询
    //var {parse2DArray} = require('views/lib/common')
    //var {currencies} = require('views/lib/common')
    //var {findPropertyGroup} = require('views/lib/common')
    var {
      world_level //与JSON的mapping一致
    } = config

    var output = {    //CouchDB所显示的内容
      id: parseInt(id),
      world_level: world_level    //output里的键应与JSON一致
    }
    var out = uppercaseFirst(output)  //将键的首字母大写

    emit(parseInt(id), out)
  }
}
```

## Elixir

1. 每一个新的项目都需要[从对应的分支中拉出开发分支](/lib/branch.md)进行编码，否则会导致代码无法push。
2. 编码时，应遵循“简单设计原则”的理念，并尽可能地做到“高内聚，低耦合”。
3. 玩家数据中，新添加的原子，务必加在所在模块的init_atoms里。
4. 仔细检查给客户端写的协议是否规范。
5. 在完成编码后，按照[服务器功能代码自查列表](/lib/服务器规则.md#服务器功能代码自查列表)进行review，如果代码尚存错误，务必及时修正。
6. 如果代码所在文件是由你创建的，需要在提交前按下**Shift + Alt + F**(VS Code)以格式化。
7. 在准备提交之前，需要解决完所有的warnning。
8. 提交代码时，提交描述需遵循[git提交描述规则](/lib/服务器规则.md#git提交描述规则)。
9. 在进行cherry-pick时，记得勾选“完成后删除”。

```txt
“简单设计原则”:
1. 通过所有的测试（Passes its tests）
2. 尽可能消除重复 (Minimizes duplication)
3. 尽可能清晰表达 (Maximizes clarity)
4. 更少的代码元素 (Has fewer elements)
```

```elixir
def init_atoms() do
  {:prop}
end
```

---
