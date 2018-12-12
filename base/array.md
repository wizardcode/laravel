## 1.对现有缓存数据进行分页返回

##### 官方默认提供对数据库进行分页查询,扩赞下对缓存数组数据进行分页查询

``` 
public function paginatorArray(Request $request)
{
    $data = Redis::hgetall(Auth::id());
    $perPage = 10;
    $currentPage = 1;
    $total = count($data);
    if ($request->has(‘page’)) {
        $current_page = $request->input(‘page’);
        $current_page = $current_page <= 0 ? 1 : $current_page;
        $current_page = $current_page > ($total/$perPage)+1 ? 1 : $current_page;
    } 
    $item = array_slice($data, ($current_page – 1) * $perPage, $perPage);    
    $paginator = new LengthAwarePaginator($item, $total, $perPage, $currentPage, [
    ‘path’ => Paginator::resolveCurrentPath(), 
    ‘pageName’ => ‘page’,
    ]);
    $userlist = $paginator->toArray()[‘data’];
    $result = compact(‘userlist’, ‘paginator’);
    return view(‘test’, [‘result’ => $result]);
}
```