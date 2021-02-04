mysql数据数据表的排序规则修改（批量修改）



在工作中同事遇到的问题，是找一种简便的方法批量修改数据表字段的排序规则，在MySQL中叫collation，和编码CHARACTER一起出现的。collation有三种级别，分辨是数据库级别，数据表级别和字段级别。

database、table、column

问题是，mysql表外键和关联的主键不能建立关系，因为排序规则的问题。

网上搜到的解决办法，都提到了修改数据表级别collation排序规则。但是我遇到的场景是数据表级别已经是utf8_unicode_ci，而字段级别是utf8_general_ci，（这里我们关心的字段类型是varchar）。

由于需要修改的字段太多了，手工修改肯定是费时费力的。自然也想到了用脚本的方式批量修改，但是发现这种通过查找MySQL信息表、过滤、拼接生成批量修改的语句太好用了，而且还能做到针对varchar类型。

SELECT CONCAT('ALTER TABLE `', table_name, '` MODIFY `', column_name, '` ', DATA_TYPE, '(', CHARACTER_MAXIMUM_LENGTH, ') CHARACTER SET UTF8 COLLATE utf8_unicode_ci', (CASE WHEN IS_NULLABLE = 'NO' THEN ' NOT NULL' ELSE '' END), ';')
 
FROM information_schema.COLUMNS
 
WHERE TABLE_SCHEMA = 'database'
 
AND DATA_TYPE = 'varchar'
 
AND
 
(
 
CHARACTER_SET_NAME != 'utf8'
 
OR
 
COLLATION_NAME != 'utf8_unicode_ci'
 
);
database需要改成实际数据库名字。需要注意的是，如果要修改的字段存在外键关系，那就要小心处理，删除外键，修改collation后再把外键关系加回来。

然后把查出来的数据再用语句用sql命令执行所有的（注意把所有|替换为空）
