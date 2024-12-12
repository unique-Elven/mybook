```sql
mysql> update typecho_contents set text = replace(text,'https://www.52ying.top','http://www.oisec.cn');
Query OK, 63 rows affected
Rows matched: 986  Changed: 63  Warnings: 0

mysql> update typecho_comments set url = replace(url,'https://www.52ying.top','http://www.oisec.cn');
Query OK, 33 rows affected
Rows matched: 87  Changed: 33  Warnings: 0

mysql> update typecho_fields set str_value = replace(str_value,'https://www.52ying.top','http://www.oisec.cn');
Query OK, 25 rows affected
Rows matched: 948  Changed: 25  Warnings: 0

mysql> update typecho_handsomecache set data = replace(data,'https://www.52ying.top','http://www.oisec.cn');
Query OK, 0 rows affected
Rows matched: 6  Changed: 0  Warnings: 0
```


