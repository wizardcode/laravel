# laravel
Laravel框架很棒，官方文档对新手不是特友好，这也可能是所谓的高门槛，致力于官方文档的细节补充。

#### 官方文档->基础功能->表单验证部分
###### 快速验证
```
$this->validate($request, [
       'title' => 'required|unique:posts|max:255',
       'author.name' => 'required',
       'author.description' => 'required',
   ]);
 validate有多个参数，适用于单个或局部验证，第一个参数为$request,不解释。第二个参数为rules，即验证规则，数组形式。
 第三个为messages，重写验证失败后的提示信息，数组形式。第四个为attributes，设置字段名对应的中文名称。使用代码如下：
 
 $this->validate($request, [
                 'title' => 'required|unique:posts|max:255',
                 'author.name' => 'required',
                 'author.description' => 'required',
             ],[
                 'required'=>':attribute为必填项',                
                 'max'=>':attribute长度超过最大限制，最大255个字节',
                 'unique'=>':attribute必须是唯一的'
             ],[
                 'title'=>'题目',
                 'author.name'=>'用户名称',
                 'author.description'=>'用户描述'
             ]);
 :attribute为占位符，用于显示当前字段名称。
 ```
 ###### 表单请求验证
 ```
 执行 php artisan make:request StoreBlogPost后,在生成的类中,生成authorize方法,默认返回值为false,若不需要判断权限认证,把返回值
 修改成true才可,否则报错信息如下
 Symfony \ Component \ HttpKernel \ Exception \ AccessDeniedHttpException
 This action is unauthorized.
 生成的类中默认有authorize和rules方法,可手动添加messages和attributes方法完成以上功能.
 
 public function authorize()
     {
         return false;
     }

     public function rules()
     {
         return [
             'title' => 'required|unique:posts|max:255',
             'author.name' => 'required',
             'author.description' => 'required',
         ];
     }
 
     public function messages()
     {
         return [
            'required'=>':attribute为必填项',                
            'max'=>':attribute长度超过最大限制，最大255个字节',
            'unique'=>':attribute必须是唯一的'
         ];
     }
     public function attributes()
     {
         return [
             'title'=>'题目',
             'author.name'=>'用户名称',
             'author.description'=>'用户描述'
         ];
     }

在控制器使用中,把依赖注入Request换成自己生成的类名,即可自动完成验证.如下:
 public function create(StoreBlogPost $request)
    {
    }
```
###### AJAX 请求 & 验证
``` 
blade模板显示错误信息文档给出了例子,Ajax请求只有文字描述,
'当我们对 AJAX 的请求中使用 validate 方法时，Laravel 并不会生成一个重定向响应，而是会生成一个包含所有验证错误信息的 JSON 响应。
这个 JSON 响应会包含一个 HTTP 状态码 422 被发送出去.'

Ajax 使用返回的错误信息如下:
<form action="/articlepicupload" method="post" id="testform">
    <input name="title">
    <input name="content">
</form>
<button onclick="saveajax()">Ajax</button>
<script>
    function saveajax() {
        $.ajax({
            url: "/articlepicupload",
            type:"post",     //请求类型
            data:$("#testform").serialize(),  //请求的数据
            dataType:"json",  //数据类型
            success: function(data){

            },
            error: function(msg) {//http状态码为422,json数据返回要在error回调函数中执行.
                var json=JSON.parse(msg.responseText);
                console.log(json.errors);
            },

        })
    }
</script>

```