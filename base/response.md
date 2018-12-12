## 1.手动解决跨域问题（ CORS ）
``` 
 可在命名空间下加入如下代码
 namespace App\Http\Controllers;
 header('Access-Control-Allow-Origin:*');
 ......
 
```
## 2.插件解决跨域问题（ CORS ）

```
GitHub插件地址:
https://github.com/barryvdh/laravel-cors
```