## 1.手动生成seed文件

```
使用seed数据填充时，先清除数据库，并重置自增ID，再进行转移，如下：
DB::connection('college_user_system@mysql')->table('sat_subjects')->truncate();
DB::connection('college_user_system@mysql')->table('sat_subjects')->insert($datas);
```
## 2.seed逆向生成器 iseed

以前苦于：

1.迁移文件回滚时会清除表中已有数据。

2.一些适合直接导入数据库的数据难以生成seeder文件，系统部署时尤其麻烦。

自从认识了iseed，这些问题已不存在

Github地址:**https://github.com/orangehill/iseed**

安装：

```
composer required orangehill/iseed --dev   生成迁移文件只在开发中使用到
使用：

php artisan iseed my_table,another_table //默认使用laravel default指定的数据库

php artisan iseed share_gifts --database=collegeData  //database为配置文件中的connect名称

php artisan iseed links --force --max=5 --exclude=created_at,updated_at

--force —— 强制覆盖已有文件；
--max —— 最多导出多少条；
--exclude —— 不包含的字段.
```