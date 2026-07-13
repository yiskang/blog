title: "Revit debugging on Visual Studio 2013"
date: 2015-05-06 17:56:25
categories:
- Tech
- BIM
tags:
- Revit
- Plugin
---

最近萬惡的微軟（M$）突然大爆發，一直開源又將 Visual Studio 2013 Professional 釋出為 Visual Studio Community 版，免費提供大家使用。用 Visual Studio 2012 也好一陣子了，所以我想換口味試看看～ :P 但裝好 2013 拿來 Debug Revit 2016 增益集（Addin）的時候，卻發現 Revit 掛掉 Crash 了！
<div style="width:600px;margin:0 auto;display:block;">

![RevitImg](images/RevitDebuggingError.jpg)

</div>
所幸拜了 Google 大神後發現了解法，只要將使用除錯器的相容模式（Debugging in  Managed Compatibility Mode）就可以解決了！以下是我所做的步㵵供大家參考：
1. 到工具（Tools） > 選項（Options），紅框所選位置：
<div style="width:300px;margin:0 auto;display:block;">

![RevitImg](images/RevitDebuggingError2.jpg)

</div>

2. 點選除錯（Debugging）項目，在點選一般（General），最後將使用相容模式（Use Managed Compatibility Mode）勾選起來，按下確認（OK）：
<div style="width:600px;margin:0 auto;display:block;">

![RevitImg](images/RevitDebuggingError3.jpg)

</div>

3. 可以開心使用 Visual Studio 2013 對 Revit 增益集進行除錯（Trace Debugging）摟～

Reference: [Getting Revit to start while debugging API apps with Visual Studio 2013](http://revitlearningclub.blogspot.tw/2015/01/getting-revit-to-start-while-debugging.html)
