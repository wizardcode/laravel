## 1.把Base64编码图片存储成图片文件

```
 $img_data=$request->img;
 $base64_str = substr($img_data, strpos($img_data, ",")+1);
 $image=base64_decode($base64_str);
 $img_name=time().'.png';
 Storage::disk('public')->put($img_name,$image); 
```