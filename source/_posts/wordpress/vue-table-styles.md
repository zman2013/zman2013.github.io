---
title: Vue table styles
tags:
  - table
  - vue
id: '344'
categories:
  - - vue
date: 2020-05-29 21:58:57
---

# Table样式
## 设置边框
```html
<style>
    .el-table--border:after,
    .el-table--group:after,
    .el-table:before {
        background-color: #cccccc;
    }

    .el-table--border,
    .el-table--group {
        border-color: #cccccc;
    }

    .el-table td,
    .el-table th.is-leaf {
        border-bottom: 1px solid #cccccc;
    }

    .el-table--border th,
    .el-table--border th.gutter:last-of-type {
        border-bottom: 1px solid #cccccc;
    }

    .el-table--border td,
    .el-table--border th {
        border-right: 1px solid #cccccc;
    }
</style>

<el-table
    :data="dataList"
    border>
</el-table>

```
## 设置排序&默认排序
```html
<el-table
    :data="dataList"
    :default-sort = "{prop: 'createtime', order: 'descending'}">
    <el-table-column
            prop="createtime"
            label="创建时间"
            sortable
            width="200">
    </el-table-column>
</el-table>
```

## 根据内容适配宽度
```html
<el-table
    :data="dataList"
    style="width:fit-content">
</el-table>
```

## 默认单元格样式
```html
<el-table :data="dataList">
    <el-table-column
            prop="name"
            label="名称"
            width="150">
    </el-table-column>
</el-table>
```

## 自定义单元格样式
```html
<el-table :data="dataList">
    <el-table-column
            prop="version"
            label="版本"
            width="150">
        <template slot-scope="scope">
            <el-input v-model="scope.row.version"></el-input>
        </template>
    </el-table-column>
</el-table>

<el-table :data="dataList">
    <el-table-column
            label="操作"
            width="100">
        <template slot-scope="scope">
            <el-button
                    size="mini"
                    type="danger"
                    @click="deleteEcu(scope.$index, scope.row)">删除</el-button>
        </template>
    </el-table-column>
</el-table>
```