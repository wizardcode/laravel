# laravel
Laravel框架很棒，官方文档对新手不是特友好，这也可能是所谓的高门槛，致力于官方文档的细节补充。

### 官方文档->基础功能->表单验证部分
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