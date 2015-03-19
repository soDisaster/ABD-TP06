# ABD-TP06

Thomas Bernard & Anne-Sophie Saint-Omer


Dans Postgresql, avec la commande EXPLAIN, examinez et expliquez les plans d’execution
des instructions suivantes :

```sql  
select * from T ;

Seq Scan on t  (cost=0.00..3.00 rows=100 width=102)  
```


select * from T where a = ‘4’ ;

Seq Scan on t  (cost=0.00..3.25 rows=1 width=102)


select T.A from T where T.A > ‘50’;

Seq Scan on t  (cost=0.00..3.25 rows=54 width=4)


select T.A, T.B from T where T.A > ‘50’;

Seq Scan on t  (cost=0.00..3.25 rows=54 width=102)


