## 1.AJAX 请求 & 验证

```
若数据验证失败,官方给出了blade模板显示错误信息的例子
当我们使用AJAX请求时,若validate方法验证失败,Laravel并不会生成一个重定向响应，而是会生成一个包含所有验证错误信息的 JSON 响应。
HTTP状态码为422.
``` 
```
Ajax 使用返回的错误信息如下:
<form action="" id="testform">
    <input name="title">
</form>
<button onclick="saveajax()">Ajax</button>
<script>
    function saveajax() {
        $.ajax({
            url: "/article/create",
            type:"post",
            data:$("#testform").serialize(),
            dataType:"json",
            success: function(data){
                console.log(data)
            },
            error: function(msg) {//http状态码为422,json数据返回要在error回调函数中执行.
                var json=JSON.parse(msg.responseText);
                console.log(json.errors);//打印出返回的错误信息
            },
        })
    }
</script>
```