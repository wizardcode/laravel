## 1.基本验证
#### 在控制器中，表单验证使用请求如下
```
可直接使用：
$this->validate($request, [
       'title' => 'required|unique:posts|max:255',
       'author.name' => 'required',
       'author.description' => 'required',
   ]);
字段名称和错误提示均使用默认
```
#### validate有多个参数，适用于单个或局部验证.
* 第一个参数为$request
* 第二个参数为rules，即验证规则，数组形式
* 第三个为messages，重写验证失败后的提示信息，数组形式
* 第四个为attributes，设置字段名对应的中文名称。
 
 ```
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
## 2.表单请求验证
```
执行 php artisan make:request StoreBlogPost后
在生成的类中,生成authorize方法,默认返回值为false,可进行权限判断用,建议使用policy进行权限验证。
把返回值修改为true即可,否则抛出如下异常.
Symfony \ Component \ HttpKernel \ Exception \ AccessDeniedHttpException
This action is unauthorized.
```

```
默认生成的类中只有authorize和rules方法,可手动添加messages和attributes方法完成以上功能.
  
  public function authorize()
  {
      return true;
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
 
表单请求验证写好后,然后在控制器使用中
通过依赖注入把Request换成自己生成的表单请求验证类,即可完成验证.如下:
  public function create(StoreBlogPost $request)
  {
      ...
  }
 ```