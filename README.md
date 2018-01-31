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
###### Cookie使用 && 记住我
```
当访问laravel网站后，系统默认在本地写入两条Cooike，名字分别为XSRF-TOKEN和laravel_session，Value通过加密存储，若value值一经修改，laravel则废弃失效
当用户在登陆时选择记住我时，成功登陆以后会下入另一条Cookie，名为remember_web_59b*，其有效期为五年，这显然是不安全的，你会发现把cookie添加到另外一台
电脑或浏览器，刷新一下就可以成功登陆，而且是一劳永逸，而真正用户不知情。使用cookie登陆本身就有安全问题，暂不考虑这些因素，在现有的cookie上进一步加强
安全。
第一步：修改remember_web_59b*的有效期。
在LoginController中，使用了trait，有“use AuthenticatesUsers;”，在其中有方法sendLoginResponse，在LoginController重写次方法，在发送到客户端之前重写过期日期。
protected function sendLoginResponse(Request $request)
    {
        $rememberTokenExpireMinutes = 43200;//一个月
        $rememberTokenName = Auth::getRecallerName();
        Cookie::queue($rememberTokenName, Cookie::get($rememberTokenName), $rememberTokenExpireMinutes);

        $request->session()->regenerate();

        $this->clearLoginAttempts($request);

        return $this->authenticated($request, $this->guard()->user())
            ?: redirect()->intended($this->redirectPath());
    }
    
第二步：登陆成功后增加一个cookie，作为登录序列，在数据库user表中增加listorder字段

protected function loginList()
 {
        $listcode = time();
        $op = User::find(Auth::id());
        $op->listorder = $listcode;
        if ($op->save() > 0) {
            $loginlist = Cookie::queue('loginlist', $listcode, 1000000);//在客户端写入cookie
            }
}
在LoginController重写redirectTo
protected function redirectTo()
 {
        $this->loginList();
        return '/home';
 }
使用密码成功登陆后可在客户端和数据库插入对应的值，通过cookie登录,不经过LoginController
第三步：在HomeController中，构造函数加入了auth中间件，故只有认证通过的用户可访问。路由home对应HomeController的index方法
public function index(Request $request)
    {
        $loginlist = $request->cookie('loginlist');
        $datebaselist = Auth::user()->listorder;
        if ($loginlist == $datebaselist) {
            if (Auth::viaRemember()) {
                $this->loginList();
                $request->session()->regenerate();
            } else {
                $this->loginList();
            }
        } else {
            $this->loginList();
            Auth::logout();
            return redirect('/login');
        }
        $result['info'] = "success";
        $result['data'] = [];
        return json_encode($result);
    }

    protected function loginList()
    {
        $listcode = time();
        $op = User::find(Auth::id());
        $op->listorder = $listcode;
        if ($op->save() > 0) {
            Cookie::queue('loginlist', $listcode, 1000000);
        }
    }
 每次认证都是重写listorder，若cookie被盗用，用户再次登录时会要求使用密码登录，同时刷新listorder和remember_web_59b*,盗取者信息失效。
    
```
#### 官方文档->综合话题->邮件发送
###### 细节问题整理
```
class SendEmail extends Mailable
{
    use Queueable, SerializesModels;
    public $order;
    public function __construct($code)
    {
        $this->order=$code;
    }
    public function build()
    {
        return $this->view('emails.email')->subject('欢迎您使用Myuniuni');
    }
}
通过构造函数赋值order，我用在随机验证码中。
在view后面通过使用subject 给邮件设置标题。
若邮件发送使用SSL，
若MAIL_HOST=smtp.mxhichina.com，则 MAIL_ENCRYPTION=ssl
若MAIL_HOST=https://smtp.mxhichina.com,则MAIL_ENCRYPTION=
```
#### 官方文档->综合话题->文件存储
###### 细节问题整理
```
if (isset($data['avatar'])) {
    $file=$request->file('avatar');
    $path = $file->storeAs('avatar', $data['id'].$file->getClientOriginalExtension());
    $member->avatar = $avatar.$path;
 }
 $file->getClientOriginalExtension() 获取文件扩展名
 $file->getClientOriginalName() 获取文件原始名称
 $file->getClientMimeType() 获取文件类型
 $file->getClientSize() 获取文件大小
 更多函数在 vendor/symfony/http-foundation/File/UploadedFile.php
 
  自定义磁盘路径
  'college' => [
             'driver' => 'local', //使用local驱动
             'root' => '../public/img/colleges', //此选项为文件存储路径,推荐使用相对路径,可以把目标文件存储到任意位置.
             'url' => env('APP_URL').'/storage',//
             'visibility' => 'public',
         ],
  root默认使用的storage_path设置路径,文件目录在/storage下,若在storage之外储存文件,推荐使用相对路径,如上.
  url设置的路径为root的符号链接,即root对应url的地址.通过访问url地址可以定向到root下.
```
#### 官方文档->综合话题->消息通知
###### 数据库通知实例
```
完成官方文档的先决条件后，打开创建好的通知类（通常存在 app/Notifications 文件夹里）
格式化数据库通知，官方实例
public function toArray($notifiable)
{
    return [
        'invoice_id' => $this->invoice->id,
        'amount' => $this->invoice->amount,
    ];
}
初次之外，其他可用代码如下：
public function via($notifiable)//设置消息频道为
      database{
        return ['database'];
    }
   
public $invoice;
public function __construct($invoice='')//invoice为控制器方法中出过来的数据，在构造函数中赋值给invoice
 {
       $this->invoice=$invoice;
  }
  
在控制器中定义方法：
 public function test()
   {   
        $invoice['id']=100;
        $invoice['amount']=200;
        $user = User::find(1);//指定用户，也可为当前登录用户。
        $user->notify(new InvoicePaid($invoice));
    }
 至此数据库消息通知完毕。
```
#### 官方文档->数据库->数据填充
###### 数据填充时笔记
```
使用seed数据填充时，使用前先清除数据库，并重置自增ID，在进行转移，如下：
       DB::connection('college_user_system@mysql')->table('sat_subjects')->truncate();
       DB::connection('college_user_system@mysql')->table('sat_subjects')->insert($datas);
```
