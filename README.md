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
#### 官方文档->综合话题->文件存储
###### 保存文件->文件上传 Base64编码接受并存储图片
```
官方给出了使用文件流的方式上传保存文件,若是图片，还可以通过base64编码保存成文件

 $data=$request->all();
 $imgdata=$imgdata['imgurl'];
 $base64_str = substr($imgdata, strpos($imgdata, ",")+1);
 $image=base64_decode($base64_str);
 $imgname=rand(1000,10000).time().'.png';
 Storage::disk('public')->put($imgname,$image);
 echo $imgname;//获取到图片名称后存储到数据库中，供以后使用。
 其他情况参照官方文档
 
 若出现跨域情况，则代码如下:
 <?php
 
 namespace App\Http\Controllers;
 header('Access-Control-Allow-Origin:*');
 ......
 
```
#### 官方文档->数据库->分页
###### 手动创建分页 对数组分页返回

``` 
public function paginatorArray(Request $request)
{
$data = Redis::hgetall(Auth::id());
$perPage = 10;
$currentPage = 1;
$total = count($data);
if ($request->has(‘page’)) {
$current_page = $request->input(‘page’);
$current_page = $current_page <= 0 ? 1 : $current_page;
$current_page = $current_page > ($total/$perPage)+1 ? 1 : $current_page;
} else {
$current_page = 1;
}
$item = array_slice($data, ($current_page – 1) * $perPage, $perPage);

$paginator = new LengthAwarePaginator($item, $total, $perPage, $currentPage, [
‘path’ => Paginator::resolveCurrentPath(), 
‘pageName’ => ‘page’,
]);
$userlist = $paginator->toArray()[‘data’];
$result = compact(‘userlist’, ‘paginator’);
return view(‘test’, [‘result’ => $result]);
}
```
#### 官方文档->安全相关->用户认证
###### 细节问题整理
```
当登录成功以后，再次访问登录或注册url，会被重定向到首页，原因是使用了中间件，如下：
app/Http/Middleware/RedirectIfAuthenticated.php中的handle方法

public function handle($request, Closure $next)
    {
        if ($this->auth->check()) {
            return redirect('/logged');
        }
        return $next($request);
    }
修改redirect的参数，可重定向到指定页面。

路由文件 web.php中 Auth:routes() 为路由键组，对应/vendor/laravel/framework/src/Illuminate/Routing/Router.php
public function auth()
    {
        // Authentication Routes...
        $this->get('login', 'Auth\LoginController@showLoginForm')->name('login');
        $this->post('login', 'Auth\LoginController@login');
        $this->post('logout', 'Auth\LoginController@logout')->name('logout');

        // Registration Routes...
        $this->get('register', 'Auth\RegisterController@showRegistrationForm')->name('register');
        $this->post('register', 'Auth\RegisterController@register');

        // Password Reset Routes...
        $this->get('password/reset', 'Auth\ForgotPasswordController@showLinkRequestForm')->name('password.request');
        $this->post('password/email', 'Auth\ForgotPasswordController@sendResetLinkEmail')->name('password.email');
        $this->get('password/reset/{token}', 'Auth\ResetPasswordController@showResetForm')->name('password.reset');
        $this->post('password/reset', 'Auth\ResetPasswordController@reset');
    }
通过请求方式get和post判断执行哪条路由。

在LoginController.php 控制器中重写login方法可以实现多个字段登录功能
   public function login(Request $request)
    {
        $username = trim($request->username);
        $password = trim($request->password);
        $remember = isset($request->remember) ? true : false;
        $patternemail = "/^[a-zA-Z0-9_-]+@[a-zA-Z0-9_-]+(\.[a-zA-Z0-9_-]+)+$/";
        $patternmobil = "/[1-9]{1}\d{10}/";
        if (preg_match($patternemail, $username)) {
            if ($this->guard()->attempt(['email' => $username, 'password' => $password], $remember)) {
                return $this->sendLoginResponse($request);
            }
        } elseif (preg_match($patternmobil, $username)) {
            if ($this->guard()->attempt(['mobile' => $username, 'password' => $password], $remember)) {
                return $this->sendLoginResponse($request);
            }
        }
        return $this->sendFailedLoginResponse($request);
    }
  attempt第一个参数为验证数组，第二参数为布尔值，为true时记录remember_token。
  执行注销后，数据库remember_token更换。
```
