---
title: element-UI 日期控件（datepicker）回显的注意事项
date: 2019-05-06 17:35:44
tags: [javascript,element-UI,Vue]
categories: 
	- [前端]
---

昨天在做一个有时间段选择的功能时，按照官方实例，如下代码

``` javascript
<template>
  <div class="block">
    <span class="demonstration">默认</span>
    <el-date-picker
      v-model="value1"
      type="date"
      value-format="YYYY-MM-DD"
      placeholder="选择日期">
    </el-date-picker>
  </div>
</template>

<script>
  export default {
    data() {
      return {
        pickerOptions: {
          disabledDate(time) {
            return time.getTime() > Date.now();
          }，
        value1: ''
      };
    }
  };
</script>
```
做完之后，要么就是回显不了，要么就是回显了之后，再次编辑时，显示的文本框里值不会变化，但是`model`里的值确实跟着变了。

1. 开始赋值的`value1`是空字符串，但是在编辑回显时，该值为一个数组，值为`Date`类型。
2. 在编辑时，用的 `Object.assgin({},row)`这种方式给`model`赋值，这个时候大概是将时间字段的类型给改变了，改成直接赋值的方式之后，该功能恢复正常。
3. 编辑时，如果没有重新选择时间，那么`model`里的该字段依然是`Date`类型的数组，根据元素配置的格式`YYYY-MM-DD`，没有值传到后台去，此时要将数组里的值主动切换成需要的时间格式，后台才能正常接收到数据。

以上是使用datepicker空间需要注意的几点。