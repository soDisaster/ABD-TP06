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


```sql

explain select tt.a, tt.t, tt.b from t join tt on t.a = tt.t ;

Hash Join  (cost=4.25..16.05 rows=267 width=103)
  Hash Cond: (tt.t = t.a)
  ->  Seq Scan on tt  (cost=0.00..8.00 rows=300 width=103)
  ->  Hash  (cost=3.00..3.00 rows=100 width=4)
        ->  Seq Scan on t  (cost=0.00..3.00 rows=100 width=4)
       
```

```sql   

explain analyze select tt.a, tt.t, tt.b from t join tt on t.a = tt.t ;

Hash Join  (cost=4.25..16.05 rows=267 width=103) (actual time=0.381..1.474 rows=267 loops=1)
  Hash Cond: (tt.t = t.a)
  ->  Seq Scan on tt  (cost=0.00..8.00 rows=300 width=103) (actual time=0.022..0.412 rows=300 loops=1)
  ->  Hash  (cost=3.00..3.00 rows=100 width=4) (actual time=0.292..0.292 rows=100 loops=1)
        ->  Seq Scan on t  (cost=0.00..3.00 rows=100 width=4) (actual time=0.022..0.152 rows=100 loops=1)
Total runtime: 1.827 ms
        
        
```



        
 

