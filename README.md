# Juegos Multijugador
(hndrk)  https://drive.google.com/drive/folders/1K4Mec_-44qFTflbwZDE5MuazgaQFzpdx?usp=drive_link
Desarrollar un videojuego es en sí mismo un desafío; Los juegos multijugador, sin embargo, añaden un conjunto completamente nuevo de problemas que hay que abordar. (Los problemas centrales son la naturaleza humana y la física)
### Arquitecturas de juegos multijugador
### Trampas
Los juegos multijugador son diferentes. En cualquier juego competitivo, un jugador que hace trampa empeora la experiencia para los demas jugadores. Como desarrollador, probablemente quieras evitar esto, ya que tiende a alejar a los jugadores de tu juego.  
No confíes en el jugador. los jugadores intentarán hacer trampa.
### Lag
Esto estropea el juego cuando se utiliza para un juego a través de Internet.  
Hablemos de física. Supongamos que estás en Bolivia, conectado a un servidor en EEUU. Eso es aproximadamente 7000 km. Nada puede viajar más rápido que la luz, ni siquiera los bytes de Internet (que en el nivel inferior son pulsos de luz, electrones en un cable u ondas electromagnéticas). La luz viaja aproximadamente a 300000 km/s, por lo que tarda 23 ms en recorrer 7000 km.

## Arquitectura cliente Servidor
### Servidores maestros y clientes simples 

![Descripción de la imagen](/images/cliente-servidor0.png "img1")  

El estado del juego lo gestiona únicamente el servidor. Los clientes envían sus acciones al servidor. El servidor actualiza el estado del juego periódicamente y luego envía el nuevo estado del juego a los clientes, quienes simplemente lo muestran en la pantalla.
### Servidores maestros y clientes predictivos
Supongamos que tenemos un retraso de 100 ms y la animación del personaje que se mueve de un cuadrado al siguiente tarda 100 ms. Usando la implementación ingenua, toda la acción tomaría 200 ms:  
Retraso de red + animación.  
![Descripción de la imagen](/images/cliente-servidor1.png "img1")   

En lugar de enviar las entradas y esperar a que el nuevo estado del juego comience a renderizarse, podemos enviar las entradas y comenzar a renderizar el resultado de esas entradas como si hubieran tenido éxito, mientras esperamos que el servidor envíe el estado "verdadero" del juego. – que la mayoría de las veces coincidirá con el estado calculado localmente:  
La animación se reproduce mientras el servidor confirma la acción.  
![Descripción de la imagen](/images/cliente-servidor2.png "img1")  
### Problemas de sincronizacion
![Descripción de la imagen](/images/cliente-servidor3.png "img1")  
Desde el punto de vista del jugador, presionaron la tecla de flecha derecha dos veces; el personaje se movió dos cuadros hacia la derecha, permaneció allí durante 50 ms, saltó un cuadro hacia la izquierda, permaneció allí durante 100 ms y saltó un cuadro hacia la derecha. Esto, por supuesto, es inaceptable.
###  Conciliación del servidor
Si el cliente simplemente aplicara esta actualización de corrección del servidor palabra por palabra, haría retroceder al cliente en el tiempo, deshaciendo por completo cualquier predicción del lado del cliente. ¿Cómo entonces resolver esto y al mismo tiempo permitir al cliente predecir el futuro?  

![Descripción de la imagen](/images/cliente-servidor4.png "img1")  
La solución es mantener un búfer circular del estado anterior de los caracteres y la entrada para el jugador local en el cliente, luego, cuando el cliente recibe una corrección del servidor, primero descarta cualquier estado almacenado en el búfer anterior al estado corregido del servidor y vuelve a reproducir. el estado comenzando desde el estado corregido hasta el momento actual "predicho" en el cliente utilizando las entradas del jugador almacenadas en el búfer circular. De hecho, el cliente "rebobina y reproduce" de forma invisible los últimos n fotogramas del movimiento del personaje del jugador local mientras mantiene fijo el resto del mundo.
### Paso de tiempo del servidor
Cliente 1 visto por el Cliente 2.  
![Descripción de la imagen](/images/cliente-servidor5.png "img1")  
No puedes simplemente actualizar las posiciones de los jugadores cuando el servidor envía datos autorizados; obtendrías jugadores que se teletransportan distancias cortas cada 100 ms, lo que hace que el juego no se pueda jugar.
#### Interpolar 
![Descripción de la imagen](/images/cliente-servidor6.png "img1")  
 La interpolación suele funcionar bastante bien. Si no es así, puedes hacer que el servidor envíe datos de movimiento más detallados con cada actualización, por ejemplo, una secuencia de segmentos rectos seguidos por el jugador, o posiciones muestreadas cada 10 ms que se ven mejor cuando se interpolan (no es necesario para enviar 10 veces más datos (ya que estás enviando deltas para movimientos pequeños, el formato en el cable se puede optimizar en gran medida para este caso particular).
### Compenzacion del Lag
Desde el punto de vista del jugador, esto tiene dos consecuencias importantes:
- El jugador se ve a sí mismo en el presente.
- El jugador ve otras entidades en el pasado.

Afortunadamente, existe una solución relativamente simple para esto, que también resulta agradable para la mayoría de los jugadores la mayor parte del tiempo (con la única excepción que se analiza a continuación).

![Descripción de la imagen](/images/cliente-servidor7.png "img1")   
Así es como funciona:

Cuando disparas, el cliente envía este evento al servidor con información completa: la marca de tiempo exacta de tu disparo y el objetivo exacto del arma.

Aquí está el paso crucial . Dado que el servidor obtiene todas las entradas con marcas de tiempo, puede reconstruir el mundo con autoridad en cualquier instante del pasado. En particular, puede reconstruir el mundo exactamente como lo veía cualquier cliente en cualquier momento.

Esto significa que el servidor puede saber exactamente qué estaba en la mira de tu arma en el instante en que disparaste. Era la posición pasada de la cabeza de tu enemigo, pero el servidor sabe que era la posición de su cabeza en tu presente.

El servidor procesa la toma en ese momento y actualiza los clientes

## Herramientas y librerias para multijugador en unity 

### Mirror
https://mirror-networking.gitbook.io/docs/

### Nomcore 
https://normcore.io/why-normcore
### UNITY Multiplayer
https://docs-multiplayer.unity3d.com/netcode/current/about/

https://unity.com/demos/small-scale-coop-sample

https://unity.com/demos/galactic-kittens

### Photon Fusion
https://www.photonengine.com/fusion#

## Demostracion en unity
Clases involucradas  

![Descripción de la imagen](/images/multiplayer1.png "img1")  

Core loop  
![Descripción de la imagen](/images/multiplayer0.png "img1")  

## Desplegar como servicio

https://unity.com/products/game-server-hosting
https://playfab.com/multiplayer/
https://heroiclabs.com/multiplayer/

### Escalar servicios

- Dokerizar
- Redis (mach y cache)
- Kubernetes  
![Descripción de la imagen](/images/multiplayer2.png "img1")  


## Referencias

https://gafferongames.com/post/introduction_to_networked_physics/

https://developer.valvesoftware.com/wiki/Latency_Compensating_Methods_in_Client/Server_In-game_Protocol_Design_and_Optimization

https://docs.unrealengine.com/udk/Three/NetworkingOverview.html

https://cloud.google.com/files/articles/google-cloud_technical-article_dedicated-server.pdf

