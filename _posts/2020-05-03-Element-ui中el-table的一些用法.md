---
layout:     post
title:      第六篇 Element-ui中el-table的一些用法
subtitle:   
date:       2020-05-03
author:     Mmdfish
header-img: 
catalog: true
tags:
    - Vue
    - ElementUI
    - El-table

---

# 关键词
Vue ElementUI el-table

# GitHub
[stockspec.vue](https://github.com/mmdfish/stockspec_vue/blob/master/stockspec/src/components/stockspec.vue)

# el-table的一些用法
[el-table](https://element.eleme.cn/#/zh-CN/component/table)的文档里有很多的例子可以参考，这里记录一些我碰到的问题

- 固定表头

文档里的方法是用

```
 <el-table height="250"></el-table>
```

但是这里有一个问题，如果希望表格占满页面，并且随着页面缩放同步变化应该怎么办。解决方法：可以用window.onresize方法来动态调整table的height

```
<el-table :height="tableHeight"></el-table>
<script>
export default {
  data() {
    return {
      tableHeight: window.innerHeight,
    };
  },
  mounted:function(){
        this.$nextTick(function () {
            this.tableHeight = window.innerHeight - 100;
            
            let self = this;
            window.onresize = function() {
                self.tableHeight = window.innerHeight - 100;
            }
        })　
    },
}
```

- 固定列

在对应的el-table-column 设置fixed属性

```
<el-table>
<el-table-column prop="code" label="代码" fixed></el-table-column>
<el-table-column prop="name" label="名字" fixed></el-table-column>
...
</el-table>

```

- 内容过长被隐藏时显示tooltip

设置show-overflow-tooltip

```
<el-table-column prop="code" label="代码" fixed show-overflow-tooltip></el-table-column>
```

- row-click怎么获取参数

row-click 调用的方法可以有三个默认的参数 row，column和event，可以从row，column里拿到想要的数据

```
<el-table :data="stockSpecData" @row-click="displayDetails">
<script>

export default {
  methods: {
    displayDetails(row, column, event) {
      if(column.label == "名字") {
        return
      }
      this.selectcode = row.code
      this.selectname = row.name
     }
  }
}
</script>
```

- 用v-for动态列渲染

如果数据的props比较多，或者想动态生成列，那可以用v-for和slot-scope来实现。
stockSpecData就是所有的spec json数据，titleData是所有想显示的列名，具体见源码。
scope.column.property代表当前列的值，scope.row[scope.column.property]是当前单元格的值。

```
<el-table :data="stockSpecData">
    <el-table-column
      v-for="(item,key) in titleData"
      :key="key"
      :prop="item.value"
      :label="item.name"
    >
      <template slot-scope="scope">
        <span>{{scope.row[scope.column.property]}}</span>
      </template>
    </el-table-column>
  </el-table>
```

以上一些都只是提供了解决方法，没有对方法进行详细介绍，可以自行搜索用法。个人感觉console.log是最实用的方法之一。
