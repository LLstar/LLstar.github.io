---
title: 利用elementUI实现一个可编辑的表格
date: 2019-10-06 15:19:48
tags:
- vue
- element-ui
categories:
- Web前端
copyright: true
---

在使用ElementUI过程中，表格的使用占了很大一部分，但是ElementUI的表格只能实现单纯的展示功能，并不能在展示表格的同时实现表格中单元格的可编辑功能，但是在业务中又需要用到可编辑的表格，于是将ElementUI中的表格和表单结合起来，实现了一个带有表单验证的可编辑表格，刚好可以满足业务需要，同时又方便了表格的使用。

<!-- more -->

代码如下：

```js
<template>
  <div style="padding: 20px;">
    <el-form ref="form" :model="formData" class="table-with-input">
      <el-table :header-cell-style="{ background: '#DCDFE6', color: '#6C6C6C' }" :data="formData.tableData" border>
        <slot name="columnheader"></slot>
        <el-table-column v-for="col in title" :key="col.prop" :label="col.label" :prop="col.prop" min-width="120px" align="center">
          <template slot-scope="scope">
            <el-form-item :prop="`tableData.${scope.$index}.${col.prop}`" :rules="col.rules">
              <el-input v-if="col.type === 'text'" v-model="scope.row[col.prop]"></el-input>
              <el-input-number
                v-if="col.type === 'money'"
                :precision="col.precision"
                :max="col.max"
                :min="col.min"
                v-model="scope.row[col.prop]"
                :controls="false"
              ></el-input-number>
            </el-form-item>
          </template>
        </el-table-column>
        <slot name="columnfooter"></slot>
      </el-table>
    </el-form>
  </div>
</template>

<script>
export default {
  data() {
    return {
      title: [
        {
          prop: 'one',
          label: '一',
          rules: [{ required: true, message: '课时不能为空', trigger: ['blur', 'change'] }],
          precision: 2,
          type: 'money',
          max: 99999,
          min: 0
        },
        {
          prop: 'two',
          label: '二',
          rules: [{ required: true, message: '课时不能为空', trigger: ['blur', 'change'] }],
          precision: 2,
          type: 'money',
          max: 99999,
          min: 0
        },
        {
          prop: 'three',
          label: '三',
          rules: [{ required: true, message: '课时不能为空', trigger: ['blur', 'change'] }],
          precision: 2,
          type: 'money',
          max: 99999,
          min: 0
        }
      ],
      formData: {
        tableData: [
          {
            one: 0,
            two: 0,
            three: 0
          }
        ]
      },
      singleColumnData: {}
    }
  },
  props: {},
  methods: {
    addCol() {
      this.formData.tableData.push(this.singleColumnData)
    },
    delCol(index) {
      this.formData.tableData.splice(index, 1)
    }
  },
  created() {
    this.title.forEach(item => {
      this.singleColumnData[item.prop] = 0
    })
    console.log('单个数据：', this.singleColumnData)
  },
  components: {},
  watch: {
    title: {
      handler(val) {
        val.forEach(item => {
          this.singleColumnData[item.prop] = 0
        })
        console.log('单个数据：', this.singleColumnData)
      },
      deep: true
    }
  },
  computed: {}
}
</script>

<style lang="scss">
.table-with-input {
  /deep/.el-input-number {
    width: 100%;
  }
  /deep/.el-table .cell,
  /deep/.el-table--border td:first-child .cell {
    padding: 0;
  }
  /deep/.el-table--enable-row-hover .el-table__body tr:hover > td {
    background-color: #fff !important;
  }
  /deep/.el-table td {
    padding: 0;
  }
  /deep/.el-form-item {
    margin-bottom: 20px;
  }
  /deep/.el-form-item__content,
  /deep/.el-input-number {
    line-height: 35px;
  }
  /deep/.el-input__inner {
    border: none;
    height: 35px;
    line-height: 35px;
    text-align: center;
    color: #3296fa;
  }
  /deep/.el-form-item__error {
    width: 100%;
    text-align: center;
  }
}
</style>
```

其实实现可编辑的表格的原理即在一个表单中添加一个表格，表格的每个单元格就是表单中的每一项，但是注意，需要将表格中的每一行数据作为一个表单的一项，这样才可以实现表单的动态校验。

表单的动态校验的原则是：将表单中每一项的prop设置为唯一的值，同时prop的值需要同v-model中绑定的值相同，我是将表格中的头数据封装为一个动态的数据，可以实现表单的复用性

`:prop="tableData.${scope.$index}.${col.prop}"`，这个属性是重点哦。

这样我们就实现了一个可编辑的表格，是不是很简单，赶快动手试一下吧。
