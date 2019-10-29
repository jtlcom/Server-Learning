# 编码规范

## 目录

* [JSON](#json)
* [JavaScript](#javascript)
* [JavaScript常用转表方法](#javascript常用转表方法)
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
      "id": "天数", //前面的名字自取，尽量采用小驼峰命名
      "worldLevel": "世界等级" //后面的与csv配置表中的属性(每一列的名字)一一对应，尽量保证顺序对应
    },
    "type": {
      "id": "int",  //与mapping的命名一致
      "worldLevel": "int"  //类型可用int, float, string
    }
  }
]   //mapping中的id会成为CouchDB中的key值，所以尽量选择csv中的索引(或者易于区分且唯一的值)作为JSON的id
```

## JavaScript

### 写法一

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
      worldLevel //与JSON的mapping一致
    } = config

    var output = {    //CouchDB所显示的内容
      id: parseInt(id),
      worldLevel: worldLevel    //output里的键应与JSON一致，尽量小驼峰
    }

    //var out = uppercaseFirst(output)  //将键的首字母大写，客户端转表时，请将此行代码打开
    emit(parseInt(id), out)
  }
}
```

### 写法二

```javascript
function (doc) {
  var [type, id] = doc._id.split(":");
  if (type === "rune_compound") {
    var {uppercaseFirst} = require('views/lib/common')
    var config = doc.value;
    var value = {};
    value["id"] = parseInt(id)

    Object.keys(config).map(function(key) {
      if (config[key] != undefined) {
        switch(key) {
          case "sweepCost": //一维数组
            value[key] = config[key].split(';').map(function(sc) {
              return parseInt(sc);
            })
            break;
          case "name": //服务器可不转的字段
            break;
          default:
              value[key] = config[key];
            break;
        }
      }
    });

    // var value = uppercaseFirst(value)
    emit(parseInt(id), value);
  }
}
```

* 第一种写法更直观的表现了转表的内容，且较为简单
* 而第二种写法可以更简便的转表，并且兼容后面新增的其他字段

### JavaScript常用转表方法

```javascript
var { findPropertyGroup, parse2DArray } = require('views/lib/common')

// format: item_1, item_2, count_1, count_2, bind_1, bind_2
var items = findPropertyGroup(config, 2, 'item_', 'count_', 'bind_') || []
items = items.map(function ([item, count, bind]) {
  item = item.split(';').map(function (it) { return parseInt(it) })
  return ['item', item, count, !!bind]
})

// format: prop_id_1..prop_id_5, prop_1..prop_5
value["props"] = findPropertyGroup(config, 5, "prop_id_", "prop_")
  .reduce(function (obj, [id, a]) {
    obj[id] = [a];
    return obj;
  }, {});

// 二维数组 format: 201999992;1#201999993;1
var parseRareProps = function (rareProps) {
  return rareProps.split('#').map(function (group) {
    return group.split(';').map(function (x) { return parseInt(x); });
  });
};

// 一维数组 format: 201999992;1
if (hintStar != undefined) {
  hintStar = hintStar.split(';').map(function (star) {
    return parseInt(star);
  });
}

// 0 -> false, !0 -> true
bind = !!bind,

// 二维数组 format: 12;30|13;00
if (timeProp) {
  var groups = timeProp.split("|");
  timeProp = groups.map(function (g) {
    [start, end] = g.split(";");
    return [start, end];
  });
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
