---
title: asp.net mvc 控制器list参数接收json传来的数组始终为空的问题
date: 2019-05-06 22:37:36
tags: [C#,MVC,JSON]
---


```c#
//控制器方法，用来接收数组参数
public ActionResult Test(List<int> ids){
      return View();
}
```

```javascript
var data = [1,2,3]

$.ajax({
  url:'@Url.Action("Test")',
  type:'POST',
  data:{ ids:data },
  traditional:true,//这句是关键
  success:function(response){
    alert('success');
  }
})
```

一开始没有js代码里注释的那一行，后台ids始终是null，加上这个配置项，后台能正常接收参数了。

关于ajax里的traditional参数的解释，可以看这篇 [https://www.jianshu.com/p/f63f538a004e](https://www.jianshu.com/p/f63f538a004e)

