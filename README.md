# ABD-TP06

Thomas Bernard & Anne-Sophie Saint-Omer


Partie 1
--------

Dans Postgresql, avec la commande EXPLAIN, examinez et expliquez les plans d’execution
des instructions suivantes :

```sql  
explain select * from T ;

Seq Scan on t  (cost=0.00..3.00 rows=100 width=102)  
```

Coût estimé du lancement : 0  
Coût total estimé : 3  
Nombre de lignes estimé en sortie par ce nœud de plan : 100  
Largeur moyenne estimée (en octets) des lignes en sortie par ce nœud de plan : 102  

Le select * renvoie toutes les lignes présentes dans la table. Le nombre de lignes de la table t étant de 100, rows = 100.    
Les champs a et b sont des varchars respectivement de taille 3 et 97. En tout ont a donc une taille de 100 octets, la largeur moyenne estimée étant de 102.  


```sql 
explain select * from T where a = ‘4’ ;

Seq Scan on t  (cost=0.00..3.25 rows=1 width=102)
```

Coût estimé du lancement : 0  
Coût total estimé : 3,25  
Nombre de lignes estimé en sortie par ce nœud de plan : 1  
Largeur moyenne estimée (en octets) des lignes en sortie par ce nœud de plan : 102  

Le select * renvoie une ligne. On cherche, en effet dans le champ "a" qui correspond à la clé primaire (elle est donc unique),  la valeur 4.  
Les champs a et b sont des varchars respectivement de taille 3 et 97. En tout ont a donc une taille de 100 octets, la largeur moyenne estimée étant de 102.  
Le coût est plus élevé que pour la requête précédente à cause de la condition where.  


```sql 
explain select T.A from T where T.A > ‘50’;

Seq Scan on t  (cost=0.00..3.25 rows=54 width=4)
Filter: (a > '50'::bpchar)
 
```
Coût estimé du lancement : 0  
Coût total estimé : 3,25  
Nombre de lignes estimé en sortie par ce nœud de plan : 54  
Largeur moyenne estimée (en octets) des lignes en sortie par ce nœud de plan : 4  

Cette requête renvoie 54 lignes, de l'id 51 à 99 mais également les id 6 à 9.  
Le champs a est un varchar de taille 3, la largeur estimée est de 4.  
Même chose que la requête précédente : le coût est plus élevé que pour la première requête à cause de la condition where.


```sql 
explain  select T.A, T.B from T where T.A > ‘50’;

Seq Scan on t  (cost=0.00..3.25 rows=54 width=102)
```

Coût estimé du lancement : 0  
Coût total estimé : 3,25  
Nombre de lignes estimé en sortie par ce nœud de plan : 54  
Largeur moyenne estimée (en octets) des lignes en sortie par ce nœud de plan : 102  

Cette requête renvoie 54 lignes, de l'id 51 à 99 mais également les id 6 à 9.  
Le champs a est un varchar de taille 3, la largeur estimée est de 4.  
Même chose que précédemment : varchar(3) + varchar(97)


Partie 2
--------

Examinez le plan d’execution du calcul de la jointure entre T et TT :

```sql
select tt.a, t.a, tt.b
from t join tt on t.a = tt.t ;

Hash Join  (cost=4.25..16.05 rows=267 width=103)
  Hash Cond: (tt.t = t.a)
  ->  Seq Scan on tt  (cost=0.00..8.00 rows=300 width=103)
  ->  Hash  (cost=3.00..3.00 rows=100 width=4)
        ->  Seq Scan on t  (cost=0.00..3.00 rows=100 width=4)
 ```
        
Le coût de lancement n'est plus 0 comme dans les requêtes précédentes, la jointure augmente ce coût.
Chaque SeqScan correspond à une table : il y a 300 lignes dans tt et 100 lignes dans t.


explain & explain analyze
-------------------------

```sql

explain select tt.a, tt.t, tt.b from t join tt on t.a = tt.t ;

Hash Join  (cost=4.25..16.05 rows=267 width=103)
  Hash Cond: (tt.t = t.a)
  ->  Seq Scan on tt  (cost=0.00..8.00 rows=300 width=103)
  ->  Hash  (cost=3.00..3.00 rows=100 width=4)
        ->  Seq Scan on t  (cost=0.00..3.00 rows=100 width=4)
       
```

Le coût de lancement n'est plus 0 comme dans les requêtes précédentes, la jointure augmente ce coût.
Chaque SeqScan correspond à une table : il y a 300 lignes dans tt et 100 lignes dans t.


```sql   

explain analyze select tt.a, tt.t, tt.b from t join tt on t.a = tt.t ;

Hash Join  (cost=4.25..16.05 rows=267 width=103) (actual time=0.381..1.474 rows=267 loops=1)
  Hash Cond: (tt.t = t.a)
  ->  Seq Scan on tt  (cost=0.00..8.00 rows=300 width=103) (actual time=0.022..0.412 rows=300 loops=1)
  ->  Hash  (cost=3.00..3.00 rows=100 width=4) (actual time=0.292..0.292 rows=100 loops=1)
        ->  Seq Scan on t  (cost=0.00..3.00 rows=100 width=4) (actual time=0.022..0.152 rows=100 loops=1)
Total runtime: 1.827 ms
        
        
```



```sql
explain select tt.a, tt.t, tt.b from tt where tt.t in (select a from t) ;

Hash Semi Join  (cost=4.25..16.01 rows=267 width=103)
  Hash Cond: (tt.t = t.a)
  ->  Seq Scan on tt  (cost=0.00..8.00 rows=300 width=103)
  ->  Hash  (cost=3.00..3.00 rows=100 width=4)
        ->  Seq Scan on t  (cost=0.00..3.00 rows=100 width=4
```



```sql
explain analyze select tt.a, tt.t, tt.b from tt where tt.t in (select a from t) ;

Hash Semi Join  (cost=4.25..16.01 rows=267 width=103) (actual time=0.385..1.463 rows=267 loops=1)
  Hash Cond: (tt.t = t.a)
  ->  Seq Scan on tt  (cost=0.00..8.00 rows=300 width=103) (actual time=0.023..0.402 rows=300 loops=1)
  ->  Hash  (cost=3.00..3.00 rows=100 width=4) (actual time=0.289..0.289 rows=100 loops=1)
        ->  Seq Scan on t  (cost=0.00..3.00 rows=100 width=4) (actual time=0.023..0.150 rows=100 loops=1)
Total runtime: 1.806 ms
```




```sql 

explain select tt.a, tt.t, tt.b from tt where tt.t is not null ;

Seq Scan on tt  (cost=0.00..8.00 rows=267 width=103)
  Filter: (t IS NOT NULL)
  
```



```sql 

explain analyze select tt.a, tt.t, tt.b from tt where tt.t is not null ;

  
  Seq Scan on tt  (cost=0.00..8.00 rows=267 width=103) (actual time=0.029..0.413 rows=267 loops=1)
  Filter: (t IS NOT NULL)
Total runtime: 0.744 ms


```

Jointures sans index 
--------------------

```sql 

explain select T.A, T.B from T join TT on T.A = TT.T ;


Hash Join  (cost=4.25..16.05 rows=267 width=102)

  Hash Cond: (tt.t = t.a)

  ->  Seq Scan on tt  (cost=0.00..8.00 rows=300 width=4)

  ->  Hash  (cost=3.00..3.00 rows=100 width=102)

        ->  Seq Scan on t  (cost=0.00..3.00 rows=100 width=102)
```

```sql

explain analyze select T.A, T.B from T join TT on T.A = TT.T ;
        
Hash Join  (cost=4.25..16.05 rows=267 width=102) (actual time=0.233..0.833 rows=267 loops=1)

  Hash Cond: (tt.t = t.a)

  ->  Seq Scan on tt  (cost=0.00..8.00 rows=300 width=4) (actual time=0.016..0.237 rows=300 loops=1)

  ->  Hash  (cost=3.00..3.00 rows=100 width=102) (actual time=0.174..0.174 rows=100 loops=1)

        ->  Seq Scan on t  (cost=0.00..3.00 rows=100 width=102) (actual time=0.012..0.081 rows=100 loops=1)

Total runtime: 1.008 ms

```
```sql

explain select distinct T.A, T.B from T join TT on T.A = TT.T ;

HashAggregate  (cost=17.38..18.38 rows=100 width=102)

  ->  Hash Join  (cost=4.25..16.05 rows=267 width=102)

        Hash Cond: (tt.t = t.a)

        ->  Seq Scan on tt  (cost=0.00..8.00 rows=300 width=4)

        ->  Hash  (cost=3.00..3.00 rows=100 width=102)

              ->  Seq Scan on t  (cost=0.00..3.00 rows=100 width=102)

```
```sql 
 
explain analyze  select distinct T.A, T.B from T join TT on T.A = TT.T ;            
              
HashAggregate  (cost=17.38..18.38 rows=100 width=102) (actual time=2.055..2.156 rows=89 loops=1)

  ->  Hash Join  (cost=4.25..16.05 rows=267 width=102) (actual time=0.372..1.521 rows=267 loops=1)

        Hash Cond: (tt.t = t.a)

        ->  Seq Scan on tt  (cost=0.00..8.00 rows=300 width=4) (actual time=0.028..0.416 rows=300 loops=1)

        ->  Hash  (cost=3.00..3.00 rows=100 width=102) (actual time=0.309..0.309 rows=100 loops=1)

              ->  Seq Scan on t  (cost=0.00..3.00 rows=100 width=102) (actual time=0.019..0.145 rows=100 loops=1)

Total runtime: 2.392 ms

```
```sql

explain select T.A, T.B from T where T.A in (select TT.T from TT)


Hash Semi Join  (cost=11.75..16.11 rows=89 width=102)

  Hash Cond: (t.a = tt.t)

  ->  Seq Scan on t  (cost=0.00..3.00 rows=100 width=102)

  ->  Hash  (cost=8.00..8.00 rows=300 width=4)

        ->  Seq Scan on tt  (cost=0.00..8.00 rows=300 width=4)

```
```sql

explain analyze select T.A, T.B from T where T.A in (select TT.T from TT)    
        
Hash Semi Join  (cost=11.75..16.11 rows=89 width=102) (actual time=0.990..1.354 rows=89 loops=1)

  Hash Cond: (t.a = tt.t)

  ->  Seq Scan on t  (cost=0.00..3.00 rows=100 width=102) (actual time=0.023..0.146 rows=100 loops=1)

  ->  Hash  (cost=8.00..8.00 rows=300 width=4) (actual time=0.901..0.901 rows=267 loops=1)

        ->  Seq Scan on tt  (cost=0.00..8.00 rows=300 width=4) (actual time=0.026..0.458 rows=300 loops=1)

Total runtime: 1.506 ms

```

```sql

explain select T.A, T.B from T where 3 = (select count(*) from TT where TT.T = T.A)

Seq Scan on t  (cost=0.00..880.25 rows=1 width=102)

  Filter: (3 = (SubPlan 1))

  SubPlan 1

    ->  Aggregate  (cost=8.76..8.77 rows=1 width=0)

          ->  Seq Scan on tt  (cost=0.00..8.75 rows=3 width=0)

                Filter: (t = $0)
   
 ```
 ```sql
 
 explain analyze select T.A, T.B from T where 3 = (select count(*) from TT where TT.T = T.A)             

 Seq Scan on t  (cost=0.00..880.25 rows=1 width=102) (actual time=0.231..9.358 rows=89 loops=1)

  Filter: (3 = (SubPlan 1))

  SubPlan 1

    ->  Aggregate  (cost=8.76..8.77 rows=1 width=0) (actual time=0.088..0.089 rows=1 loops=100)

          ->  Seq Scan on tt  (cost=0.00..8.75 rows=3 width=0) (actual time=0.044..0.080 rows=3 loops=100)

                Filter: (t = $0)

Total runtime: 9.638 ms

```


Jointures avec index 
--------------------

```sql 

explain select T.A, T.B from T join TT on T.A = TT.T ;


Hash Join  (cost=4.25..16.05 rows=267 width=102)

  Hash Cond: (tt.t = t.a)

  ->  Seq Scan on tt  (cost=0.00..8.00 rows=300 width=4)

  ->  Hash  (cost=3.00..3.00 rows=100 width=102)

        ->  Seq Scan on t  (cost=0.00..3.00 rows=100 width=102)       
        
```

```sql

explain analyze select T.A, T.B from T join TT on T.A = TT.T ;
        
Hash Join  (cost=4.25..16.05 rows=267 width=102) (actual time=0.428..1.521 rows=267 loops=1)

  Hash Cond: (tt.t = t.a)

  ->  Seq Scan on tt  (cost=0.00..8.00 rows=300 width=4) (actual time=0.028..0.411 rows=300 loops=1)

  ->  Hash  (cost=3.00..3.00 rows=100 width=102) (actual time=0.341..0.341 rows=100 loops=1)

        ->  Seq Scan on t  (cost=0.00..3.00 rows=100 width=102) (actual time=0.020..0.167 rows=100 loops=1)

Total runtime: 1.854 ms

```
```sql

explain select distinct T.A, T.B from T join TT on T.A = TT.T ;

HashAggregate  (cost=17.38..18.38 rows=100 width=102)

  ->  Hash Join  (cost=4.25..16.05 rows=267 width=102)

        Hash Cond: (tt.t = t.a)

        ->  Seq Scan on tt  (cost=0.00..8.00 rows=300 width=4)

        ->  Hash  (cost=3.00..3.00 rows=100 width=102)

              ->  Seq Scan on t  (cost=0.00..3.00 rows=100 width=102)
```
```sql 
 
explain analyze  select distinct T.A, T.B from T join TT on T.A = TT.T ;       

HashAggregate  (cost=17.38..18.38 rows=100 width=102) (actual time=2.086..2.188 rows=89 loops=1)

  ->  Hash Join  (cost=4.25..16.05 rows=267 width=102) (actual time=0.387..1.544 rows=267 loops=1)

        Hash Cond: (tt.t = t.a)

        ->  Seq Scan on tt  (cost=0.00..8.00 rows=300 width=4) (actual time=0.024..0.446 rows=300 loops=1)

        ->  Hash  (cost=3.00..3.00 rows=100 width=102) (actual time=0.331..0.331 rows=100 loops=1)

              ->  Seq Scan on t  (cost=0.00..3.00 rows=100 width=102) (actual time=0.019..0.159 rows=100 loops=1)

Total runtime: 2.417 ms

```
```sql

explain select T.A, T.B from T where T.A in (select TT.T from TT);


Hash Semi Join  (cost=11.75..16.11 rows=89 width=102)

  Hash Cond: (t.a = tt.t)

  ->  Seq Scan on t  (cost=0.00..3.00 rows=100 width=102)

  ->  Hash  (cost=8.00..8.00 rows=300 width=4)

        ->  Seq Scan on tt  (cost=0.00..8.00 rows=300 width=4)

```
```sql

explain analyze select T.A, T.B from T where T.A in (select TT.T from TT)    
        
Hash Semi Join  (cost=11.75..16.11 rows=89 width=102) (actual time=0.934..1.296 rows=89 loops=1)

  Hash Cond: (t.a = tt.t)

  ->  Seq Scan on t  (cost=0.00..3.00 rows=100 width=102) (actual time=0.022..0.144 rows=100 loops=1)

  ->  Hash  (cost=8.00..8.00 rows=300 width=4) (actual time=0.843..0.843 rows=267 loops=1)

        ->  Seq Scan on tt  (cost=0.00..8.00 rows=300 width=4) (actual time=0.022..0.432 rows=300 loops=1)

Total runtime: 1.447 ms
```

```sql

explain select T.A, T.B from T where 3 = (select count(*) from TT where TT.T = T.A)

Seq Scan on t  (cost=0.00..880.25 rows=1 width=102)

  Filter: (3 = (SubPlan 1))

  SubPlan 1

    ->  Aggregate  (cost=8.76..8.77 rows=1 width=0)

          ->  Seq Scan on tt  (cost=0.00..8.75 rows=3 width=0)

                Filter: (t = $0)
   
 ```
 ```sql
 
 explain analyze select T.A, T.B from T where 3 = (select count(*) from TT where TT.T = T.A)             
Seq Scan on t  (cost=0.00..880.25 rows=1 width=102) (actual time=0.242..8.346 rows=89 loops=1)

  Filter: (3 = (SubPlan 1))

  SubPlan 1

    ->  Aggregate  (cost=8.76..8.77 rows=1 width=0) (actual time=0.078..0.079 rows=1 loops=100)

          ->  Seq Scan on tt  (cost=0.00..8.75 rows=3 width=0) (actual time=0.037..0.071 rows=3 loops=100)

                Filter: (t = $0)

Total runtime: 8.559 ms

```


  



        
 

