# Grupo 3 - Módulo 2: Proveedores y paquetes

**Su parte:** situaciones marcadas A y las trampas (T). Para cada una: elijan el patrón, escriban por qué y un patrón que descartaron con el motivo. Si ningún patrón hace falta, escriban “ninguno” y fundamenten.

## Escenario C: Proveedores externos

Combinen 3 o 4 patrones para resolver todo lo siguiente:

- Integrar tres aerolíneas con APIs distintas y no modificables.
- El viajero elige cómo comparar (precio, duración) en el momento de buscar.
- Una sola operación “buscar y reservar” oculta consultar, bloquear y cobrar.
- El conector correcto se crea a partir de un nombre.

**Entregable:** un dibujo de cajas y flechas con los patrones elegidos y una frase por cada uno. El cambio de último momento llega más tarde.

## Situaciones

| N° | Situación | Patrón elegido | ¿Por qué? / Descartado y por qué |
|---|---|---|---|
| P6 | Una aerolínea nos da una API que no podemos modificar: `buscarVuelos(origen, destino)` y devuelve XML. La app espera `buscar(criterio)` y trabaja con objetos propios. | **Adapter** | **¿Por qué?** Un adaptador implementa la interfaz que espera la app, por dentro llama a la API y traduce el XML a nuestros objetos. **Descartado: Proxy.** También es un intermediario, pero tiene la misma cara (interfaz) que el objeto real. Acá el problema es justamente que las interfaces son distintas: hay que traducir, no interponer. |
| P8 | Un paquete contiene vuelos, hoteles, excursiones y otros paquetes más chicos. Hay que calcular el precio total de cualquier nivel con el mismo código. | **Composite** | **¿Por qué?** Es una estructura de árbol (todo-parte) con grupos dentro de grupos. Vuelo, hotel y excursión son hojas y Paquete es el contenedor; todos tienen `precio()`. El paquete suma el `precio()` de sus hijos sin preguntar si son hojas o paquetes, así el mismo código sirve en cualquier nivel. **Descartado: Iterator.** Sirve para recorrer una colección sin conocer cómo está guardada, pero no hace que un elemento suelto y un grupo se traten igual ni resuelve el anidamiento de paquetes. |
| P10 | Reservar exige verificar disponibilidad, bloquear el asiento, cobrar, emitir el voucher y mandar un mail. La web y la app móvil repiten los cinco pasos. | **Facade** | **¿Por qué?** Dos clientes (web y app) duplican la misma secuencia de llamadas a cinco subsistemas distintos. Una fachada `ServicioReservas.reservar()` esconde los cinco pasos detrás de una sola operación: se elimina la duplicación y, si cambia un paso, se cambia en un solo lugar. **Descartado: State.** State aplica cuando el objeto se comporta distinto según su estado (p. ej., cancelar una reserva pendiente vs. pagada). Acá los cinco pasos son siempre los mismos y en el mismo orden: no hay estados ni transiciones. |
| T2 (T) | Mostrar 150 minutos como “2 h 30 min”. | **Ninguno** | **¿Por qué?** Es una cuenta simple (`150 // 60 = 2 h`, `150 % 60 = 30 min`). No hay nada que varíe, que se extienda ni subsistemas que esconder. Alcanza con una función utilitaria; meter un patrón agregaría clases sin resolver ningún problema. **Descartado: Strategy.** Solo valdría la pena si hubiera varios formatos intercambiables que el usuario elige (“2 h 30 min”, “02:30”, “150 min”). Con un solo formato no hay estrategias entre las cuales elegir. |

## Escenario C: patrones elegidos

### Patrones elegidos

- **Adapter (punto 9):** cada aerolínea tiene un adaptador que implementa nuestra interfaz `ConectorAerolinea` y traduce su API (XML, JSON, etc.) a nuestros objetos `Vuelo`.
- **Strategy (punto 10):** cada criterio de comparación (`PorPrecio`, `PorDuracion`) es una estrategia intercambiable que el viajero elige al buscar.
- **Facade (punto 11):** `ServicioReservas.buscarYReservar()` esconde consultar → bloquear → cobrar detrás de una sola operación.
- **Factory Method (punto 12):** `FabricaConectores.crear("aerolineaA")` devuelve el adaptador correcto a partir del nombre, sin que el cliente haga `new` de clases concretas.

### Dibujo (cajas y flechas)

- Flecha llena: usa o llama.
- Flecha punteada con triángulo: implementa la interfaz.
- Flecha punteada “crea”: la fábrica instancia adaptadores.

### Dos patrones que se combinan y cómo

**Factory Method + Adapter:** la fábrica no crea cualquier cosa, crea adaptadores. Recibe un nombre (`"aerolineaB"`) y devuelve un `AdaptadorB` tipado como `ConectorAerolinea`. Así la fachada nunca conoce las clases concretas ni las APIs externas: pide un conector por nombre y le habla siempre con la misma interfaz.

También se combinan **Facade + Strategy**: la fachada recibe la estrategia elegida por el viajero y la aplica a los resultados sin saber cuál es.

### Un patrón que descartaron y por qué

**Proxy:** también es un intermediario entre la app y la API de la aerolínea, pero mantiene la misma interfaz que el objeto real. Acá el problema es que las interfaces son distintas: hay que traducir (Adapter), no solo interponer.

### Cuando llegue el giro: ¿qué cambia y qué no se toca?

Respuestas preparadas para los giros más probables:

- **Nueva aerolínea:** cambia → un `AdaptadorD` nuevo y una línea en la fábrica. No se toca → fachada, estrategias, cliente y otros adaptadores.
- **Nuevo criterio (escalas, horario):** cambia → una clase `PorEscalas` nueva. No se toca → fachada, adaptadores y fábrica.
- **Una aerolínea cambia su API:** cambia → solo su adaptador. No se toca → todo lo demás.
- **Nuevo paso (voucher, mail):** cambia → solo el método de la fachada. No se toca → el cliente (web/app).

**Idea a defender:** principio abierto/cerrado (SOLID): se agrega código nuevo sin modificar el que ya funcionaba.
