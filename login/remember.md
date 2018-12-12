## 1.Cookie使用 && 记住我

```
当访问laravel网站后，系统默认在本地写入两条Cooike，名字分别为XSRF-TOKEN和laravel_session,Value通过加密存储，若value值一经修改，laravel则废弃失效.
当用户在登陆时选择记住我时，成功登陆以后会写入另一条Cookie，名为remember_web*，
其有效期为五年，这显然是不安全的，你会发现把cookie添加到另外一台电脑或浏览器中便可成功登陆，而且是一劳永逸,真正用户不知情。
使用cookie登陆本身就有安全问题，若暂不考虑这些因素，在现有的cookie上进一步加强安全。
```
```
第一步:修改remember_web*的有效期。
在LoginController中，使用了trait，有“use AuthenticatesUsers;”
在其中有方法sendLoginResponse，在LoginController重写此方法，在发送到客户端之前重写过期日期。

protected function sendLoginResponse(Request $request)
{
    $rememberTokenExpireMinutes = 43200; //一个月
    $rememberTokenName = Auth::getRecallerName();
    Cookie::queue($rememberTokenName, Cookie::get($rememberTokenName), $rememberTokenExpireMinutes);
    $request->session()->regenerate();
    $this->clearLoginAttempts($request);
    return $this->authenticated($request, $this->guard()->user())
        ?: redirect()->intended($this->redirectPath());
}
```
```    
第二步:登陆成功后增加一个cookie，作为登录序列，在数据库user表中增加list_order字段

protected function loginList()
{
    $list_code = time();
    $op = Auth::user();
    $op->list_order = $list_code;
    $op->save()
    Cookie::queue('login_list', $list_code, 1000000);//在客户端写入cookie
}
```
```
第三步:在LoginController重写redirectTo,在成功跳转之前把cookie写入到客户端

protected function redirectTo()
{
    $this->loginList();
    return '/home';
}
```
```
第四步：在HomeController中，构造函数加入了auth中间件，故只有认证通过的用户可访问,路由home对应HomeController的index方法.
使用密码成功登陆后可在客户端和数据库插入对应的值，通过cookie登录,不经过LoginController
public function index(Request $request)
{
    $login_list = $request->cookie('login_list');
    $datebase_list = Auth::user()->list_order;
    if ($login_list == $datebase_list) {
        if (Auth::viaRemember()) {
            $request->session()->regenerate();
        }
        $this->loginList();
    } else {
        Auth::logout();
        return redirect('/login');
    }
    return redirect('/myhome');;
}

每次认证都是重写list_order，若cookie被盗用，用户再次登录时listorder不匹配,要求使用密码登录，同时刷新list_order和remember_web*,盗取者信息失效。
    
```