### 模拟Ajax请求

#### 先看laravel对请求是否为Ajax的判断

```
public function ajax()
{
    return $this->isXmlHttpRequest();
}
public function isXmlHttpRequest()
{
   return 'XMLHttpRequest' == $this->headers->get('X-Requested-With');
}
```
#### 模拟Ajax请求

```
namespace App\Http\Controllers;

use GuzzleHttp\Client;
use Illuminate\Http\Request;

class TestController extends Controller
{
    protected $client;

    public function __construct()
    {
        $this->client = new Client([
            'headers' => [
                "X-Requested-With" => "XMLHttpRequest",
            ],
        ]);
    }

    public function testAjax(Request $request)
    {
        if ($request->ajax()) {
            return "Ajax请求";
        } else {
            return "非Ajax请求";
        }
    }

    public function get()
    {
        $result = $this->client->get("http://local.myuniuni.com/test123");
        $data = $result->getBody()->getContents();
        echo $data;
    }
  ```
    
#### 无论实际使用任何请求方式，后端均判断为Ajax请求，输出“Ajax请求”
