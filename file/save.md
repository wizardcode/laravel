## 1.文件存储&&获取扩展信息

```
if ($request->avatar) {
    $file=$request->file('avatar');
    $path = $file->storeAs('avatar', $data['id'].$file->getClientOriginalExtension());
}
$file->getClientOriginalExtension() 获取文件扩展名
$file->getClientOriginalName() 获取文件原始名称
$file->getClientMimeType() 获取文件类型
$file->getClientSize() 获取文件大小
更多函数在 vendor/symfony/http-foundation/File/UploadedFile.php
```