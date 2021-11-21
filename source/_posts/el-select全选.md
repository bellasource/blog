---
title: el-select全选
date: 2020/04/11
tags: ["element"]
---

{% blockquote %}
el-select 官方不支持下拉全选功能，整理了两种方案，实现全选
{% endblockquote %}

### 方案一

增加全选项，并添加原生点击事件；监听选项值的变化

```html
<el-select
  v-model="value"
  multiple
  placeholder="请选择"
  @change="handleChange"
  @remove-tag="removeTag"
>
  <el-option label="全部" value="all" @click.native="handleClickAll" />
  <el-option
    v-for="item in options"
    :key="item.value"
    :label="item.label"
    :value="item.value"
  />
</el-select>
```

```javascript
handleClickAll() {
  if (this.value.length < this.options.length) {
    this.value = [...this.options.map((item) => item.value), "all"];
  } else {
    this.value = [];
  }
},
handleChange(val) {
  if (!val.includes("all") && val.length === this.options.length) {
    this.value.unshift("all");
  } else if (val.includes("all") && val.length - 1 < this.options.length) {
    this.value = this.value.filter((item) => item !== "all");
  }
},
removeTag(val) {
  if (val === "all") {
    this.value = [];
  }
},
```

### 方案二

增加全选项，监听选中的 value 值的变化

```html
<el-select v-model="value" multiple collapse-tags>
  <el-option label="全选" value="all" />
  <el-option
    v-for="item in options"
    :key="item.value"
    :label="item.label"
    :value="item.value"
  />
</el-select>
```

```javascript
//监听value
watch: {
  value: {
    handler(newValue = [], oldValue = []) {
      const isNewValueAll = newValue.includes('all')
      const isOldValueAll = oldValue.includes('all')
      const newValueLen = newValue.length
      const optionLen = this.options.length
      if((isNewValueAll && !isOldValueAll)){
        // 点击全选
        this.value = ['all', ...this.childOptions.map(({ value }) => value)];
      }else if(!isNewValueAll && isOldValueAll && newValueLen === optionLen){
        // 当所有选项全部点着，再次点击全选，清空所有选项
        this.value = [];
      }else if(isNewValueAll && isOldValueAll && newValueLen === optionLen){
        // 全选状态下，点击除全选的其他项
        this.value = newValue.filter((item) => item !== 'all');
      }else if(!isNewValueAll && newValueLen === optionLen){
        // 除all，其他选项被全选的时候触发
        this.value = ['all', ...this.childOptions.map(({ value }) => value)];
      }
    },
  },
},
```
