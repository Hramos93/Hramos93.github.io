---
layout: post
title: "Contando con SQL"
subtitle: "Cuántos clientes tiene un negocio"
background: '/img/posts/count_sql/SQL.PNG'
---


## Contando en SQL 


Pensemos un segundo que tenemos un tabla de clientes, nueva o de la cuál unimos con otra sucursal ect, y  el CEO o los directivos de una empresa necesitan saber cuántos clientes nuevos se han registrado en el negocio

La mision aquí es simple, Contar,Contar es uno de los problemas con los que te enfrentas típicamente en análisis de datos, probablemente suene sencillo, contar es una de las primeras habilidades que aprendemos desde pequeños, sin embargo la cuestión no es que tengas la habilidad sino como transformas la habilidad en conocimiento, es decir no basta con saber usar una herramienta sino en cómo te planteas un problema para poder resolver con la herramienta.

Imaginemos la siguiente tabla:


![](/img/posts/count_sql/tabla.PNG)

Formular el problema es lo primero que necesitamos, para formular debemos comprender, y desarrollar un método o pipeline de resolución.

Queremos saber cuántos usuarios nuevos se han agregado cada día.
Tenemos las columnas 
- create_at: corresponde a la fecha en la que el usuario fue agregado
- deleted_at: que corresponde la fecha de cuando el usuario fue borrado
- id: identificador unico
- merged_at: se refiere a usuarios que fueron añadidos desde otra base de datos
- parent_user_id : el id correspondiente a la tabla donde se encontraba el usuario antes

Qué hacer:
- Necesitamos saber cuántos clientes hay registrados por día.
- Conocer quienes han sido eliminados por día
- Determinar que usuarios se refieren a un mismo usuario con otra id por ejenplo  que proveniente desde otra DB o tabla por día.
- Substraer los usuarios eliminados (clientes que ya no pertenecen a la BD) y los usuarios con el mismo id.



Veamos primero qué relación hay entre las columnas de identificación:

~~~~sql
SELECT
id 
,parent_user_id, 
merged_at,
FROM dsv1069.users 
ORDER BY parent_user_id ASC
~~~~

Podemos observar que hay usuarios  que en una primera entrada(parent_user_id)  comparten la identificación en la tabla y hay otros usuarios que no, estos ids que no son iguales serán excluidos ya que se encuentran igualmente en la tabla y equivalen a los mismos usuarios solo que para un momento, o desde otra tabla fueron identificados de forma distinta.

![](/img/posts/count_sql/review.PNG)

Además de eso debemos tener solo en cuenta filas donde la columna delete_at sea Null, es decir, si en la fila delete_at no hay una fecha(día el cual fue eliminado) quiere decir que los usuarios existen o no han sido eliminados, igualmente si en la columna parent_user_id es null, quiere decir que ese registro posee un único id del cual podemos confiar.

Veamos cómo podemos escribir esto en todo esto en SQL


~~~~sql
SELECT 
    date(created_at) AS day,
    COUNT(*) AS users
FROM 
    dsv1069.users
WHERE 
    deleted_at IS NULL
    AND
        (id <> parent_user_id OR parent_user_id IS NULL)
GROUP BY 
    date(created_at)
~~~~

![](/img/posts/count_sql/figure1.PNG)


Esto nos da una idea de cuantos clientes se agregan por día, pero no está tomando en cuenta los clientes eliminados ese día,
para tomar en cuenta los clientes eliminados deberemos seleccionar los registros con valores presentes en la columna deleted_at, contarlos y agruparlos, todo esto teniendo en cuenta el total de clientes que tenemos por día, para luego restarlo.


Una explicación sencilla: 
### Query
- Columna con clientes totales por día
- Columna con clientes borrados por dia
#### Subqueries
     - Total de clientes agregados por día
     - Total de clientes eliminados por día


Y hacer una substracción.

~~~~sql
SELECT 
new.day, 
new.new_users_added,
deleted.deleted_users AS deleted_users
FROM(
-- TOTAL USERS
SELECT 
  date(created_at) AS day,
  COUNT(*) AS new_users_added
  FROM 
  dsv1069.users
  GROUP BY 
  date(created_at)
  ) new 
  -- DELETE USERS
  LEFT JOIN 
    (SELECT 
    date(deleted_at) AS day,
    COUNT(*) AS deleted_users 
    FROM 
    dsv1069.users 
    WHERE deleted_at IS NOT NULL 
    GROUP BY 
    date(deleted_at)
    ) deleted
  ON deleted.day = new.day

~~~~

![](/img/posts/count_sql/figure2.PNG)

Ok, ahora esto va teniendo más sentido.

Ahora contemos  a los usuarios con distinta id pero que ya estan en la tabla

~~~~sql
SELECT 
new.day, 
new.new_users_added,
COALESCE(deleted.deleted_users,0) AS deleted_users,
COALESCE(merged.merged_users,0) AS merged_users,
(new.new_users_added - COALESCE(deleted.deleted_users,0) - COALESCE(merged.merged_users,0)) AS net_added_users
FROM(
SELECT 
  date(created_at) AS day,
  COUNT(*) AS new_users_added
  FROM 
  dsv1069.users
  GROUP BY 
  date(created_at)
  ) new 
  -- DELETE USERS
  LEFT JOIN 
    (SELECT 
    date(deleted_at) AS day,
    COUNT(*) AS deleted_users 
    FROM 
    dsv1069.users 
    WHERE deleted_at IS NOT NULL 
    GROUP BY 
    date(deleted_at)
    ) deleted
  ON deleted.day = new.day
  -- MERGED USERS
  LEFT JOIN 
  (SELECT 
  date(merged_at) AS day,
  COUNT(*) AS merged_users
  FROM dsv1069.users
  WHERE 
  id <> parent_user_id 
  AND 
  parent_user_id IS NOT NULL 
  GROUP BY 
  date(merged_at)
  ) merged
  ON merged.day = new.day 
  ORDER BY new_users_added DESC
~~~~

![](/img/posts/count_sql/final_table.PNG)

![](/img/posts/count_sql/graph_final.PNG)

Bueno, esto es todo, espero te hayas divertido y encontrado sentido al hecho de que contar no solo se trata de conocer las cantidades sino que con un poco de astucia puedes desarollar operaciones bastante divertidas, hasta pronto.


```python

```
