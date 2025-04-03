# Análisis exploratorio de datos (EDA):

# 1. Información sobre la actividad de vuelo de los clientes `df_vuelo`:

### 1.1. Situación inicial `df_vuelo`, tipos de datos:

![alt text](images/image.png)


- `Loyalty Number`: int64, todo ok aparentemente.

- `year`: columna int64 y podría ser de tipo datatime. Posibilidad de juntar con `month`

- `month`: int64 y podría ser de tipo datatime, juntar con `year`.

- `flights booked`,`Flights with Companions`, `Total Flights`: todo ok a priori, son int64. Además, parece que:
*vuelos booked + vuelos con acompañantes = total vuelos.*

- `Distance`: int64, parece que ok. Ver posibles relaciones, porque es probable que a más distancia, más puntos (`points accumulated`).

- `Points Accumulated`: float64. Hay clientes que tienen por ejemplo, "474.12" puntos.

- `Points Redeemed`: int64. todo ok a priori.

- `Dollar Cost Points Redeemed`:  int64. Ver posibles relaciones directas con ` Points Redeemed`.


### 1.2. Situacion inicial nulos (%) `df_vuelo`: 

![alt text](images/image-1.png)

No obstante, sí tenemos muchos "0". Como son todas las columnas numéricas, estos ceros podrían significar o bien que ese dato == 0 o bien que no se dispone de ese dato (==Dato nulo). Vamos a investigar cuántos "0" tienen las columnas:

```python
df_vuelo['Total Flights'].value_counts().sort_values(ascending=False)

#pongo ascending=False porque el 0 es el que más frecuencia tiene y así me sale primero.
```

Lista de columnas que comparten número de datos == 0:

- 197992 datos == "0". Posible relación porque si no has viajado, no has realizado reservas y no tienes puntos acumulados.
    - `Total Flights`
    - `Flights Booked`
    - `Points Accumulated`


- 381443 datos == "0":
    - `Points Redeemed`: los valores únicos siguen el siguiente rango: 447 a 876.

    - `Dollar Cost Points Redeemed`:  Parece que cuando  == 0, se corresponde a los puntos canjeados (se han canjeado 0 ptos).

- 296887 == "0":
    - `Flights with Companions`: rango de datos desde 1 a 11 personas como acompañantes.



### 1.3. Análisis duplicados en `df_vuelo`:
Eliminamos los duplicados de filas:

```python
#Observamos si hay duplicados:

df_vuelo[df_vuelo.duplicated(subset='Loyalty Number',keep=False)]

#output: 3712 rows × 10 columns
```
![alt text](images/image-3.png)


Parece que hay 3712 /  2 duplicados = `1856` ==> los eliminamos.


`OJO!`👀
Tras hacer todos estos cambios, nos damos cuenta de que parece que podríamos agrupar por cliente (Loyalty Number), Year y Month,  porque todas las filas podrían sumarse:

![alt text](images/image20.png)

--> lo hacemos:
```python
df_vuelo_reducido = 
df_vuelo.groupby(['Loyalty Number', 'Year', 'Month'])[['Flights Booked','Flights with Companions', 'Total Flights', 'Distance','Points Accumulated', 'Points Redeemed', 'Dollar Cost Points Redeemed']].sum().reset_index()
```
--> reducimos 2072 filas


### 1.4. Estadísticas básicas iniciales `df_vuelos`:
Cogemos sólo las columnas que nos interesan (dejo fuera de momento las fechas y el Loyalty number):

![alt text](images/image-5.png)

Ya veremos más adelante, pero por lo general, parece que:
-  la mitad de los clientes tienen un sólo vuelo realizado con la compañía, 
- suelen volar solos (más de la mitad vuelan solos) 
- Vistas las estadísticas anteriores, más de la mitad, no suelen gastar puntos (tampoco acumulan demasiados porque los clientes hacen de media sólo 1 reserva).



# 2. Información sobre el perfil detallado de los clientes `df_cliente`:

### 2.1. Situación inicial, tipos de datos `df_cliente`:
![alt text](images/image-4.png)

- `Loyalty Number`: ok, int64

- `Country`, `Province`,`City`, `Postal Code` : object, ok

- `Gender`: object, ok. Unique: ['Female', 'Male'].

- `Education`: object, ok. Unique: ['Bachelor', 'College', 'Master', 'High School or Below', 'Doctor']

- `Salary`: float64, OJO hay salarios negativos, que he visto en las estadísticas el valor mínimo:

```python

df_cliente[df_cliente['Salary'] < 0].shape
#output: (20,16) 
#--> hay 20 salarios negativos... 
# Consultar con la empresa si esto puede ser un error y haya que ponerlos todos en valor absoluto, y en caso afirmativo, cambiarlo a valor absoluto.

```


- `Marital Status`: object, ok. Unique: ['Married', 'Divorced', 'Single']

- `Loyalty Card`: object, ok. Unique: ['Star', 'Aurora', 'Nova'] #niveles de categoría dentro del programa de lealtad

- `CLV` (Customer Lifetime Value): float64. Valor que el cliente aporta a la empresa. *Parece una columna interesante para las estadísticas!*

- `Enrollment Type`: object. Tipo de inscripción del cliente, unique: ['Standard', '2018 Promotion']

- `Enrollment Year`, `Enrollment Month` : int64. Cambiar a tipo datetime.

- `Cancellation Year`, `Cancellation Month`: float64. Cambiar a tipo datetime.


### 2.2. Situacion inicial nulos (en %) `df_cliente`:

![alt text](images/image-6.png)
- `Salary`: en principio, hay muy pocos nulos, lo dejaremos así porque no es representativo. Se podría hacer una imputación, pero no afectaría demasiado a la población porque es muy pequeño porcentaje. Se pueden sacar conclusiones robustas con ese % de nulos.



### 2.3. Análisis duplicados `df_cliente`:
No hay duplicados en este df.

### 2.4 Estadísticas iniciales básicas `df_cliente`:

Conclusiones iniciales, a profundizar:

#### Variables categóricas `df_cliente`:

![alt text](images/image-7.png)

- todos los clientes son de Canadá y la ciudad más frecuente es Toronto.

- Podría ser interesante analizar el código Postal "V6E 3D9". Incluirlo como Next Steps para una posible campaña publicitaria.

- Las mujeres son más frecuentes que los hombres.

- Los clientes más frecuentes: casados, con licenciatura, con tarjeta Loyalty Star y programa Standard.

#### Variables numéricas `df_cliente`:

Seleccionamos sólo Salary y CLV porque los meses todavía no nos dicen gran cosa, todavía tenemos que transformarlos:

![alt text](images/image-10.png)

- `Salary`: hay salarios negativos... Como comentado anterior, ver si tras hablar con la aerolínea podemos pasarlos todos a valor absoluto.

- `CLV` (valor cliente para la empresa): en principio, por la desviación típica, parece que hay mucha dispersión en este dato.. La media y la mediana están muy alejadas.


# 3. Información sobre la actividad de vuelo de los clientes `df` (tras el merge):

### 2.1. Situacion duplicados `df`:
No debería haber, pero comprobar.

### 2.2. Situacion nulos (en %) `df`:

- `Cancellation Year`, `Cancellation Month`: antes los nulos no preocupaban porque sólo había una línea por cliente, pero tras el merge, tenemos filas duplicadas, comprobar qué pasa con estos nulos, ¿los dejamos así?

- 

