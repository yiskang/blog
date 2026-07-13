title: "Use Atlassian Source Tree embedded git in Windows"
date: 2015-05-06 18:01:09
categories:
- Tech
- Windows
tags:
- Git
---

前些日子因為要在 Mac OS X 上面安裝 Git 的圖形化管理程式，接觸了 Atlassian SourceTree 跟 Github for Mac，但由於工作上有需要使用 Atlassian Bitbucket，所以最後我選擇了 SourceTree。最近因為一些原因工作用的 PC 被還原了，什麼都沒有了，只好一一重裝！裝好 SourceTree 後，我發現 SourceTree 本身就有將 Git for Windows 的執行檔包起來，所以不用另外再裝 Git，但如果想直接使用 SourceTree 內建的 Git 工具要怎麼做呢？以下是我所做的步驟供大家參考：

1. Atlassian SourceTree 附帶 Git 執行檔的位置在：
```path
%USERPROFILE%\AppData\Local\Atlassian\SourceTree\git_local\bin
```
2. 將 Git 執行檔的位置加到使用者環境變數（User Environment Variables）：
<div style="width:300px;margin:0 auto;display:block;">

![GitImg](images/GitPath.jpg)

</div>
3. 重開命令提示字元（Command Prompt）或是 PowerShell，就可以開心使用摟～


Reference: [How to run a git command as a custom action?](https://answers.atlassian.com/questions/245850/how-to-run-a-git-command-as-a-custom-action)
