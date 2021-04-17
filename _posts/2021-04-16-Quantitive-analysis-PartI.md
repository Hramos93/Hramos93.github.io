---
layout: post
title: "Introducción al análisis financiero con Python"
subtitle: "Obteniendo Data desde yahoo Finance"
background: '/img/posts/market/01.jpg'
---

Entre algunos análisis que miro frecuentemente están los análisis de mercado, específicamente quantitive analysis(QA), es una rama muy amigable del análisis de mercado,dado que me gusta programar  he decido realizar una series de guías de cómo utilizar e interpretar estás las herramientas estadísticas que tenemos a disposición para medir, modelar y comprender el comportamiento del mercado a través de programación en python. 
Por ahora me planteo hacer tres guías de iniciación, serán muy sencillas, son los primeros pasos que se realizan para los estudios de mercado.


- Obteniendo Data desde yahoo Finance 
- Herramientas básicas estadísticas (Desviación, Normalidad, skewness, Kurtosis y más)
- Introducción a la Optimización de Portafolio

Estás son las primeras guías que tengo en mente, algo que espero que tomen en cuenta es que utilizaré conceptos en inglés, por ejemplo no suelo decir "acciones", sino shares o equities, igual intentare traducir lo más que pueda, pero el material con el que estudio es en inglés y no conozco todas las traducciones, igual en su defecto estaré explicando cada concepto.


Por último y más importante, cualquier comentario o opinión emitida aquí no es una recomendación ni sugerencia en cuanto a administrar su portafolio de inversión, cada quien es responsable de las desición que tome,sin más dilación comencemos.


Inciemos extrayendo data de algunas shares  del mercado venezolano, esto para explicar como normalizo los precios al precio USD$ dado la inestabilidad que hay con el Bs actualmente, esto  facilitara los cálculos si tienes intención de participar en el mercado venezolano, sin embargo funciona para cualquier mercado que se encuentre en yahoo finance.

Para extraer la data del mercado venezolano, se hará uso de la API de yahoo finance donde se encuentran registro de los precios históricos y actuales de cada de una de las acciones con las que se quiera comerciar,por otro lado, a la fecha de hoy venezuela sufre un periodo de crisis economica que mantiene una inestabilidad en la moneda nacional,lo cual dificulta los cálculos a nivel histórico dado a la variación de precios, es decir, el precio hace dos años dista mucho del precio actual debido a la inflación.
Para resolver problema anterior se realizará un ajuste del precio en Bs al precio del USD$ , para conseguir historico cambiario del par USD/VEF, descargar el csv desde el 2018 [USD/VEF](https://www.investing.com/currencies/usd-vef-historical-data).

recuerda instalar las siguientes librerias

- !pip install pandas #libreria que ofrece un framework amigable para el manejo de tablas
- !pip numpy  #libreria para el manejo de operaciones matematicas
- !pip matplotlib  #libreria para graficar 
- !pip yfinance #API de yahoo finance.


```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import yfinance  as yf
```

El primero paso que daremos consistira en hacer lectura de la archivo descargado desde investing,
la estructura de este archivo es basicamente un fecha, el precio promedio, el precio con que abre el mercado,
su valor más alto, su valor más bajo <<estos tres son iguales por alguna extraña razon>> y el porcentaje de cambio.



```python
pd.read_csv('assets/USD_VES.csv', header=0, index_col=0, parse_dates=True )
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Price</th>
      <th>Open</th>
      <th>High</th>
      <th>Low</th>
      <th>Change %</th>
    </tr>
    <tr>
      <th>Date</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2021-04-15</th>
      <td>2,375,839.2500</td>
      <td>2,378,816.5000</td>
      <td>2,378,816.5000</td>
      <td>2,378,816.5000</td>
      <td>2.88%</td>
    </tr>
    <tr>
      <th>2021-04-14</th>
      <td>2,309,375.0000</td>
      <td>2,312,269.0000</td>
      <td>2,312,269.0000</td>
      <td>2,312,269.0000</td>
      <td>-0.85%</td>
    </tr>
    <tr>
      <th>2021-04-13</th>
      <td>2,329,146.0000</td>
      <td>2,332,064.7500</td>
      <td>2,332,064.7500</td>
      <td>2,332,064.7500</td>
      <td>1.94%</td>
    </tr>
    <tr>
      <th>2021-04-12</th>
      <td>2,284,762.0000</td>
      <td>2,287,625.2500</td>
      <td>2,287,625.2500</td>
      <td>2,287,625.2500</td>
      <td>2.44%</td>
    </tr>
    <tr>
      <th>2021-04-09</th>
      <td>2,230,234.2500</td>
      <td>2,233,029.0000</td>
      <td>2,233,029.0000</td>
      <td>2,233,029.0000</td>
      <td>4.36%</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>2018-01-05</th>
      <td>0.0001</td>
      <td>0.0001</td>
      <td>0.0001</td>
      <td>0.0001</td>
      <td>0.00%</td>
    </tr>
    <tr>
      <th>2018-01-04</th>
      <td>0.0001</td>
      <td>0.0001</td>
      <td>0.0001</td>
      <td>0.0001</td>
      <td>0.00%</td>
    </tr>
    <tr>
      <th>2018-01-03</th>
      <td>0.0001</td>
      <td>0.0001</td>
      <td>0.0001</td>
      <td>0.0001</td>
      <td>0.00%</td>
    </tr>
    <tr>
      <th>2018-01-02</th>
      <td>0.0001</td>
      <td>0.0001</td>
      <td>0.0001</td>
      <td>0.0001</td>
      <td>0.00%</td>
    </tr>
    <tr>
      <th>2018-01-01</th>
      <td>0.0001</td>
      <td>0.0001</td>
      <td>0.0001</td>
      <td>0.0001</td>
      <td>0.00%</td>
    </tr>
  </tbody>
</table>
<p>836 rows × 5 columns</p>
</div>




Una configuración amigable para la lectura es, tomar la primera fila como los header que son los nombres de la columna, 
colocar la fecha como indice y indicarle que el indice es una fecha, ademas de eso indicaremos que solo necesitamos 
la columna de precio, ordenamos y renombramos la columna con un nombre más claro.



```python
usdVEF = pd.read_csv('assets/USD_VES.csv', header=0, index_col=0, parse_dates=True )['Price']
usdVEF = pd.DataFrame(usdVEF.sort_index()) #ordenamos de manera ascendente
usdVEF.columns = ['Bs/USD$']  #renombramos la columna
usdVEF.tail()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Bs/USD$</th>
    </tr>
    <tr>
      <th>Date</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2021-04-09</th>
      <td>2,230,234.2500</td>
    </tr>
    <tr>
      <th>2021-04-12</th>
      <td>2,284,762.0000</td>
    </tr>
    <tr>
      <th>2021-04-13</th>
      <td>2,329,146.0000</td>
    </tr>
    <tr>
      <th>2021-04-14</th>
      <td>2,309,375.0000</td>
    </tr>
    <tr>
      <th>2021-04-15</th>
      <td>2,375,839.2500</td>
    </tr>
  </tbody>
</table>
</div>





Ahora comencemos con lo bueno, haremos lectura de unas pocas acciones para entender el funcionamiento.
- 'BNC.CR' 
- 'MVZ-B.CR'
- 'CIE.CR' 
- 'FNC.CR'

Primero creemos un DataFrame vacio, basicamente lo que haremos se empaquetar todos los datos allí, solo ten presente eso.
Haremos lectura con de nuestras shares con la funcion Ticker, esta se encuentra dentro de la liberia de yahoo finance, esta funcion prepara los datos para ser leidos es como "oye prepara la share X que va ser leida", 
una vez preparada hacemos lectura deacuerdo al periodo que queremos, por lo general mientras más datos tengas mejor, pero por ahora igual que con el par VEF/USD, aqui necesitamo solo el precio, y tomaremos solo 3 años de datos.

Esto funciona para un shares, pero ahora queremos varios por lo requeriremos hacer el recorrido varias veces,
además de eso queremos meter todo junto en una misma tabla, o concatenar todos los datos.

Meteré toda esta información en una función, una función nos facilitará mucho la vida, solo tendremos
que cambiar la lista de shares de la que queremos hacer lectura y función hará el mismo trabajo.


```python
equities = ['PTN.CR', 'MVZ-B.CR', 'FNC.CR']
def data(shares):
    df = pd.DataFrame()
    for i in shares:
        read = yf.Ticker(i)
        hist = read.history(period="3y")['Close']
        data = pd.DataFrame(hist)
        data['Close'].columns = [i]
        data = data.loc[~data.index.duplicated(keep='first')]
        df = pd.concat([df, data], axis=1)
        
    df.columns = shares
    return df
```


```python
df = data(equities)
df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>PTN.CR</th>
      <th>MVZ-B.CR</th>
      <th>FNC.CR</th>
    </tr>
    <tr>
      <th>Date</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2018-04-16</th>
      <td>0.139857</td>
      <td>24.606991</td>
      <td>0.286</td>
    </tr>
    <tr>
      <th>2018-04-17</th>
      <td>0.139857</td>
      <td>24.606991</td>
      <td>0.340</td>
    </tr>
    <tr>
      <th>2018-04-18</th>
      <td>0.134862</td>
      <td>29.528389</td>
      <td>0.400</td>
    </tr>
    <tr>
      <th>2018-04-19</th>
      <td>0.134862</td>
      <td>29.528389</td>
      <td>0.400</td>
    </tr>
    <tr>
      <th>2018-04-20</th>
      <td>0.139857</td>
      <td>29.528389</td>
      <td>0.400</td>
    </tr>
  </tbody>
</table>
</div>



Ahora que tenemos los datos del precio del dolar y el precio de nuestras shares, uniremos todo para ajustar los precios de las shares.

Luego de unir los datos, limpearemos un poco nuestro jardin, el formato de bs/USD$ son interpreados como string, asi que modificaremos eso para poder trabajar con el valor, luego estandarizamos, divimos el precio de nuestra accion al precio del dolar del mismo día.
Por último, tenemos una diferencia en las fechas entre los dataframe, la fecha de VEF/USD esta desde el enero 2018, mientra que alguna de nuestras shares solo tenemos data a partir de abril, esas diferencia entre fechas es rellenada con valores NaN, estos valores indican la ausencia de datos, removeremos todas esas filas ya que no presetan información de los precios.



```python
dfUSD = usdVEF.join(df)
dfUSD['Bs/USD$'] = dfUSD['Bs/USD$'].str.replace(',','').astype(np.float64)
dfUSD = dfUSD.iloc[:,1:].div(dfUSD['Bs/USD$'],axis=0).dropna(how='all')
```


```python
dfUSD.describe()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>PTN.CR</th>
      <th>MVZ-B.CR</th>
      <th>FNC.CR</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>744.000000</td>
      <td>744.000000</td>
      <td>744.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>0.121614</td>
      <td>11.345256</td>
      <td>0.455251</td>
    </tr>
    <tr>
      <th>std</th>
      <td>0.215303</td>
      <td>24.504737</td>
      <td>0.374862</td>
    </tr>
    <tr>
      <th>min</th>
      <td>0.018838</td>
      <td>0.907388</td>
      <td>0.032763</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>0.032810</td>
      <td>2.236953</td>
      <td>0.222391</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>0.041672</td>
      <td>3.191995</td>
      <td>0.311285</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>0.110085</td>
      <td>4.872743</td>
      <td>0.600751</td>
    </tr>
    <tr>
      <th>max</th>
      <td>1.083583</td>
      <td>137.110170</td>
      <td>1.972814</td>
    </tr>
  </tbody>
</table>
</div>



El primer concepto que revisaremos será el performance de la shares, en español se refiere al rendimiento o retorno.
Recordemos que el retorno para un tiempo $$t$$ a un tiempo $${t+1}$$ esta dado por :

$$ R_{t,t+1} = \frac{P_{t+1}-P_{t}}{P_{t}}$$

una alternativa muy común que veo en excel es usar esta.

$$ R_{t,t+1} = \frac{P_{t+1}}{P_{t}} - 1$$

Secillamente le estamos pidiendo que compare el precio con un precio anterior y nos muestre el resultado en terminos de porcentaje.

Es una función sencilla pero antes de que intentes algo como este código,

```python

dfUSD.iloc[1:]/dfUSD.iloc[:-1].values - 1

```

Recuerda necesitas forzar un lag en la serie de tiempo porque trabajas con un tiempo adelante, y de esa forma aunque el cálculo esta bien planteado acorde al punto de referencia(tiempo) no estará bien ordenado, te invito a probar como ejercicio.

Para resolver este problema de periodos python tiene la función [shift()](https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.shift.html), esta función permite mover un paso adelante los registros, intetemos!.


```python
FNC = dfUSD['FNC.CR']/dfUSD['FNC.CR'].shift(1) -1
FNC.head()
```




    Date
    2018-04-16         NaN
    2018-04-17    0.188811
    2018-04-18    0.176471
    2018-04-19    0.000000
    2018-04-20   -0.110995
    Name: FNC.CR, dtype: float64



Entendiendo como funciona el cálculo, y como puedes hacerlo por ti mismo, ahora te dare el camino fácil, es la función de python [ptc_change()](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.pct_change.html)

<cite>"Percentage change between the current and a prior element".</cite> esta es la definición que ofrece la documentación, justo lo que queremos, veamos si funciona.


```python
FNCR = dfUSD['FNC.CR'].pct_change()
FNCR.head()
```




    Date
    2018-04-16         NaN
    2018-04-17    0.188811
    2018-04-18    0.176471
    2018-04-19    0.000000
    2018-04-20   -0.110995
    Name: FNC.CR, dtype: float64

Perfecto! tenemos exactamente el mismo resultado que plantea la formula, podemos confiar en esta declaración, ahora veamos como podemos representar esos datos.


```python
FNCR.plot(figsize=(12,7))
plt.grid(alpha=0.5)
plt.title('Performance FNC 2018-2021')
plt.show()
```


    
![jpg](/img/posts/market/04.png)
    
Esta figura nos muestra cosas muy interesante como por ejemplo cerca a septiembre del 2018 hubo una caida basta fuerte casí del 100% del precio, curiosamente en agosto del 2018 ocurrio una reconversión monetaria en Venezuela, también podemos ver como el precio comienza  a tomar ajustes y como comienza a ofrecer retornos  entre +- 25%, pero para ver el rendimiento de una share es preferible usar un histograma, veamos el rendimiento de esta forma. 


```python
FNCR['2021':'2021'].hist(bins=30, figsize=(7,5))
plt.title('performance distribution FNC')
plt.show()
```
   
![jpg](/img/posts/market/2.png)
    
Ahorap odemos ver que para la fecha 2018-2021 ha tenido retornos negativos y positivos en frecuencias similares, sin embargo los negativos ocupan un porción relativamente mayor, además vemos que aunque con poca probabilidades se pueden obtener un rendimiento de hasta el 20% pero igual manera se puede tener perdidas de casi el 20%.

Por último quiero que miremos algo muy interesante, y es el indexado que seleccionamos al comienzo, esto también será mucha más utilidad más adelante cuando necesitemos comparar $$t$$ con $${t-n}$$, no profundizare en eso ahora, simplemente quiero mostrar cómo con el índice podemos elegir los periodos de estudio, es algo muy sencillo,tenemos que seleccionar el periodo que queremos evaluar como se muestra en el  siguiente código.


```python
data['fecha inicio':'fecha final']....
```

Veamos si funciona.


```python
FNCR['2021':'2021'].plot(figsize=(12,7))
plt.grid(alpha=0.5)
plt.title('Performance FNC año 2021')
plt.show()
```


    
![png](/img/posts/market/3.png)
    


Esto puedes usarlo para graficar, en tablas y cálculos, una genialidad.

Bueno esto es todo por ahora, espero que haya sido muy ilustrativo y que hayas aprendido bastante, sin más que decirte espero verte en la segunda parte de este mini-turorial.

