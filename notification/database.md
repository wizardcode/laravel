## 1.数据库消息通知

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