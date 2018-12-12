## 1.成功登录后的重定向

```
登录成功以后，再次访问登录或注册url，会被重定向到首页，原因是使用了中间件，如下：
app/Http/Middleware/RedirectIfAuthenticated.php中的handle方法

public function handle($request, Closure $next)
{
    if ($this->auth->check()) {
        return redirect('/logged');
    }
    return $next($request);
}
修改redirect的参数，可重定向到指定页面。
```
## 2.默认Auth路由组中的路由
```
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
```
## 3.实现多字段登录
```
在LoginController.php 控制器中重写login方法可以实现多个字段登录功能

public function login(RequestLogin $request)
{    
    $password = $request->password;
    $remember = $request->remember ?? false;
  
    if ($request->has("email")) {
        $email = $request->email;
        if ($this->guard()->attempt(['email' => $email, 'password' => $password], $remember)) {
            return $this->sendLoginResponse($request);
        }
    } elseif ($request->has("mobile")) {
        $mobile = $request->mobile;
        if ($this->guard()->attempt(['mobile' => $mobile, 'password' => $password], $remember)) {
            return $this->sendLoginResponse($request);
        }
    }
    return $this->sendFailedLoginResponse($request);
}
attempt方法第一个参数为验证数组，第二参数为布尔值，为true时记录remember_token。
执行注销后，数据库remember_token字段自动更换。

```