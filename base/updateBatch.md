## 1.批量更新数据
##### 封装生成update set case when then 的Mysql语句
``` 
updateBatch方法共三个参数
第一个:$tableName 必填,要插入的数据库表名称,字符串类型.
第二个:$multipleData 必填,要批量更新的数据, 二维数据类型.
第三个:$referenceColumn 可选,若不为空,根据该传入的字段进行更新.若为空,则使用传入数据的第一列

实例:
$status = $commonController->updateBatch("college_overview_rankings", $insert_data, "id");
```
```
use Illuminate\Support\Facades\DB;

public function updateBatch($tableName = "", $multipleData = array(), $referenceColumn = "")
    {
        if ($tableName && !empty($multipleData)) {
            $updateColumn = array_keys($multipleData[0]);
            if (empty($referenceColumn)) {
                $referenceColumn = $updateColumn[0]; //e.g id
                unset($updateColumn[0]);
            } else {
                $referenceColumn = $referenceColumn; //e.g id
                $updateColumn = array_diff($updateColumn, [$referenceColumn]);
            }

            $whereIn = "";

            $q = "UPDATE " . $tableName . " SET ";
            foreach ($updateColumn as $uColumn) {
                $q .= $uColumn . " = CASE ";

                foreach ($multipleData as $data) {
                    $q .= "WHEN " . $referenceColumn . " = " . $data[$referenceColumn] . " THEN '" . $data[$uColumn] . "' ";
                }
                $q .= "ELSE " . $uColumn . " END, ";
            }
            foreach ($multipleData as $data) {
                $whereIn .= "'" . $data[$referenceColumn] . "', ";
            }
            $q = rtrim($q, ", ") . " WHERE " . $referenceColumn . " IN (" . rtrim($whereIn, ', ') . ")";

            // Update
            return DB::update(DB::raw($q));

        } else {
            return false;
        }
    }
```