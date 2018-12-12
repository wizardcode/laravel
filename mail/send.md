## 1.邮件发送配置

```
若邮件发送使用SSL
若MAIL_HOST=smtp.mxhichina.com，则 MAIL_ENCRYPTION=ssl
若MAIL_HOST=https://smtp.mxhichina.com,则MAIL_ENCRYPTION=

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
```