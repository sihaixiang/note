```javascript
/*
批量执行、取消GEE导出任务 
首先需要生成导出任务，并显示了运行按钮；然后按F12进入控制台，将下面这些脚本粘贴到其中，然后按输入；最后所有任务自动启动（任务多的话，网络不好可能暂时没反应等一下就好了）。亲测支持Firefox和Chrome
*/
 
//运行所有task
function runTaskList() {
    var tasklist = document.getElementsByClassName('awaiting-user-config');
    for (var i = 0; i < tasklist.length; i++)
        tasklist[i].children[2].click();
}
function confirmAll() {
    var ok = document.getElementsByClassName('goog-buttonset-default goog-buttonset-action');
    for (var i = 0; i < ok.length; i++)
        ok[i].click();
}
runTaskList();
confirmAll();
 
 
//取消所有task
function runTaskList() {
    var tasklist = document.getElementsByClassName('indicator');
    for (var i = 0; i < tasklist.length; i++)
        tasklist[i].click();
}
 
function confirmAll() {
    var ok = document.getElementsByClassName('goog-buttonset-default goog-buttonset-action');
    for (var i = 0; i < ok.length; i++)
        ok[i].click();
}
runTaskList();
confirmAll();
 
```

