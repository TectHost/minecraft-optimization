# Guía de optimización de servidores de Minecraft

Nota para los usuarios que están en vanilla, Fabric o Spigot (o cualquier cosa por debajo de Paper) - vaya a su server.properties y cambie `sync-chunk-writes` a `false`. Esta opción está forzosamente en false en Paper y sus forks, pero en otras implementaciones de servidor necesitas cambiarla a false manualmente. Esto permite al servidor guardar trozos fuera del hilo principal, disminuyendo la carga en el bucle principal.

Guía para la versión 1.19. Algunas cosas todavía se pueden aplicar a 1.15 - 1.18.

(Traducción de https://github.com/YouHaveTrouble/minecraft-optimization)

Basado en [esta guía](https://www.spigotmc.org/threads/guide-server-optimization%E2%9A%A1.283181/) and other sources (all of them are linked throughout the guide when relevant).

Utilice la tabla de contenidos situada más arriba (junto a `README.md`) para navegar fácilmente por esta guía.

# Intro
Nunca habrá una guía que te dé resultados perfectos. Cada servidor tiene sus propias necesidades y límites sobre cuánto puedes o estás dispuesto a sacrificar. Juguetear con las opciones para ajustarlas a las necesidades de tu servidor es de lo que se trata. Esta guía sólo pretende ayudarte a entender qué opciones tienen impacto en el rendimiento y qué cambian exactamente. Si crees que has encontrado información incorrecta en esta guía, eres libre de abrir una incidencia o hacer un pull request para corregirla.

# Preparativos

## Server JAR
La elección del software de servidor puede marcar una gran diferencia en el rendimiento y las posibilidades de la API. Actualmente existen múltiples JARs de servidor populares y viables, pero también hay algunos de los que deberías alejarte por varias razones.

Las mejores opciones recomendadas:
* [Paper](https://github.com/PaperMC/Paper) - El software de servidor más popular que pretende mejorar el rendimiento al tiempo que corrige las incoherencias mecánicas y de juego.
* [Pufferfish](https://github.com/pufferfish-gg/Pufferfish) - Fork de paper que pretende mejorar aún más el rendimiento de los servidores.
* [Purpur](https://github.com/PurpurMC/Purpur) - Fork de Pufferfish que se centra en las prestaciones y la libertad de personalización.

Usted debe mantenerse alejado de:
* Cualquier JAR de servidor de pago que afirme async cualquier cosa - 99,99% de posibilidades de ser una estafa.
* Bukkit/CraftBukkit/Spigot - Extremadamente anticuado en términos de rendimiento comparado con otro software de servidor al que tenga acceso.
* Cualquier plugin/software que habilite/deshabilite/recargue plugins en tiempo de ejecución. Vea [esta sección](#plugins-enablingdisabling-other-plugins) para entender por qué.
* Muchos forks posteriores a Pufferfish o Purpur encontrarán inestabilidad y otros problemas. Si busca más ganancias de rendimiento, optimice su servidor o invierta en un fork privado personal.

## Pregenerar el mapa
La pregeneración de mapas, gracias a varias optimizaciones a la generación de trozos añadidas a lo largo de los años, ahora sólo es útil en servidores con CPUs terribles, de un solo hilo o limitadas. Aunque, la pregeneración es comúnmente usada para generar chunks para plugins de mapas del mundo como Pl3xMap o Dynmap.

Si todavía quieres pregenerar el mundo, puedes usar un plugin como [Chunky](https://github.com/pop4959/Chunky) para hacerlo. Asegúrate de establecer un borde del mundo para que tus jugadores no generen nuevos trozos. Ten en cuenta que la pregeneración puede tardar horas dependiendo del radio que establezcas en el plugin de pregeneración. Ten en cuenta que con Paper y superiores tus tps no se verán afectados por la carga de chunks, pero la velocidad de carga de chunks puede ralentizarse significativamente cuando la cpu de tu servidor está sobrecargada.

Es importante recordar que el overworld, el nether y el end tienen fronteras separadas que necesitan ser configuradas para cada mundo. La dimensión del nether es 8 veces más pequeña que la del overworld (si no se modifica con un datapack), así que si configuras mal el tamaño, tus jugadores pueden acabar fuera de la frontera del mundo.

**Asegúrate de configurar un borde de mundo vainilla (`/worldborder set [diameter]`), ya que limita ciertas funcionalidades como el rango de búsqueda de mapas del tesoro que pueden causar picos de lag.**

# Configuraciones

## Red

### [server.properties]

#### network-compression-threshold

`Buen valor inicial: 256`

Permite establecer el límite del tamaño de un paquete antes de que el servidor intente comprimirlo. Establecerlo más alto puede ahorrar algunos recursos de CPU a costa de ancho de banda, y establecerlo a -1 lo desactiva. Establecerlo más alto también puede perjudicar a los clientes con conexiones de red más lentas. Si su servidor está en una red con un proxy o en la misma máquina (con menos de 2 ms de ping), desactivar esto (-1) será beneficioso, ya que las velocidades de la red interna normalmente pueden manejar el tráfico adicional sin comprimir.

### [purpur.yml]

#### use-alternate-keepalive

`Good starting value: true`

Puedes habilitar el sistema de keepalive alternativo de Purpur para que los jugadores con mala conexión no se desconecten tan a menudo. Tiene incompatibilidad conocida con TCPShield.

> Habilitando esta opción se envía un paquete keepalive una vez por segundo a un jugador, y sólo se desconecta si no se responde a ninguno de ellos en 30 segundos. Respondiendo a cualquiera de ellos en cualquier orden mantendrá al jugador conectado. Es decir, no expulsará a tus jugadores porque un paquete se haya caído en algún momento.  
~ https://purpurmc.org/docs/Configuration/#use-alternate-keepalive

---

## Chunks

### [server.properties]

#### simulation-distance

`Buen valor inicial: 4`

La distancia de simulación es la distancia en trozos alrededor del jugador que el servidor marcará. Esencialmente es la distancia desde el jugador a la que sucederán las cosas. Esto incluye hornos de fundición, cultivos y árboles jóvenes creciendo, etc. Esta es una opción que querrás poner baja a propósito, en algún lugar alrededor de `3` o `4`, debido a la existencia de `view-distance`. Esto permite cargar más trozos sin marcarlos. Esto permite a los jugadores ver más lejos sin el mismo impacto en el rendimiento.

#### view-distance

`Buen valor inicial: 7`

Esta es la distancia en trozos que se enviará a los jugadores, similar a la distancia de vista sin pulsar del papel.

La distancia total de la vista será igual al mayor valor entre `distancia-simulación` y `distancia-vista`. Por ejemplo, si la distancia de simulación es 4, y la distancia de vista es 12, la distancia total enviada al cliente será de 12 trozos.

### [spigot.yml]

#### view-distance

`Buen valor inicial: default`

Este valor sobrescribe server.properties uno si no se establece en `default`. Usted debe mantener por defecto para tener tanto la simulación y la distancia de vista en un solo lugar para facilitar la gestión.

### [paper-world configuration]

#### delay-chunk-unloads-by

`Buen valor inicial: 10s`

Esta opción le permite configurar cuánto tiempo permanecerán cargados los chunks después de que un jugador se vaya. Esto ayuda a no cargar y descargar constantemente los mismos trozos cuando un jugador va y viene. Valores demasiado altos pueden resultar en que se carguen demasiados chunks a la vez. En áreas que son frecuentemente teletransportadas y cargadas, considera mantener el área permanentemente cargada. Esto será más ligero para tu servidor que cargar y descargar trozos constantemente.

#### max-auto-save-chunks-per-tick

`Buen valor inicial: 8`

Le permite ralentizar el ahorro incremental de mundo repartiendo la tarea en el tiempo aún más para un mejor rendimiento medio. Es posible que desee establecer este más alto que `8` con más de 20-30 jugadores. Si el guardado incremental no puede terminar a tiempo, bukkit guardará automáticamente los trozos sobrantes de una vez y comenzará el proceso de nuevo.

#### prevent-moving-into-unloaded-chunks

`Buen valor inicial: true`

Cuando está activada, evita que los jugadores se muevan hacia trozos no cargados y causen cargas de sincronización que atascan el hilo principal causando lag. La probabilidad de que un jugador tropiece con un trozo descargado es mayor cuanto menor sea la distancia de visión.

#### entity-per-chunk-save-limit

```
Buenos valores iniciales:

    area_effect_cloud: 8
    arrow: 16
    dragon_fireball: 3
    egg: 8
    ender_pearl: 8
    experience_bottle: 3
    experience_orb: 16
    eye_of_ender: 8
    fireball: 8
    firework_rocket: 8
    llama_spit: 3
    potion: 8
    shulker_bullet: 8
    small_fireball: 8
    snowball: 8
    spectral_arrow: 16
    trident: 16
    wither_skull: 4
```

Con la ayuda de esta entrada puedes establecer límites a cuantas entidades de un tipo específico pueden ser guardadas. Usted debe proporcionar un límite para cada proyectil por lo menos para evitar problemas con cantidades masivas de proyectiles que se guardan y el servidor se bloquea en la carga que. Puedes poner cualquier id de entidad aquí, mira la wiki de minecraft para encontrar IDs de entidades. Por favor, ajuste el límite a su gusto. Valor sugerido para todos los proyectiles es de alrededor de `10`. También puede agregar otras entidades por sus nombres de tipo a esa lista. Esta opción de configuración no está diseñada para evitar que los jugadores hagan grandes granjas de mobs.

### [pufferfish.yml]

#### max-loads-per-projectile

`Buen valor inicial: 8`

Especifica la cantidad máxima de trozos que un proyectil puede cargar durante su vida. Si se reduce, se reducirán las cargas de trozos causadas por proyectiles de entidad, pero podrían producirse problemas con tridentes, enderpearls, etc.

---

## Mobs

### [bukkit.yml]

#### spawn-limits

```
Buenos valores iniciales:

    monsters: 20
    animals: 5
    water-animals: 2
    water-ambient: 2
    water-underground-creature: 3
    axolotls: 3
    ambient: 1
```

La matemática de limitar mobs es `[playercount] * [limit]`, donde "playercount" es la cantidad actual de jugadores en el servidor. Lógicamente, cuanto menor sea el número, menos mobs verás. `per-player-mob-spawn` aplica un límite adicional a esto, asegurando que los mobs se distribuyen equitativamente entre los jugadores. Reducir esto es un arma de doble filo; sí, tu servidor tiene menos trabajo que hacer, pero en algunos modos de juego los mobs de desove natural son una gran parte de la jugabilidad. Puedes llegar a 20 o menos si ajustas `mob-spawn-range` adecuadamente. Si ajustas `mob-spawn-range` más bajo parecerá que hay más mobs alrededor de cada jugador. Si estás usando Paper, puedes establecer los límites de mobs por mundo en [paper-world configuration].

#### ticks-per

```
Buenos valores iniciales:

    monster-spawns: 10
    animal-spawns: 400
    water-spawns: 400
    water-ambient-spawns: 400
    water-underground-creature-spawns: 400
    axolotl-spawns: 400
    ambient-spawns: 400
```

Esta opción establece la frecuencia (en ticks) con la que el servidor intenta desovar ciertas entidades vivas. Los mobs de agua/ambiente no necesitan aparecer cada tick ya que no suelen morir tan rápido. En cuanto a los monstruos: Aumentar ligeramente el tiempo entre desovaciones no debería afectar a la tasa de desovaciones incluso en granjas de monstruos. En la mayoría de los casos todos los valores de esta opción deberían ser superiores a "1". Establecer este valor más alto también permite a su servidor para hacer frente mejor a las zonas donde el desove mob está desactivado.

### [spigot.yml]

#### mob-spawn-range

`Buen valor inicial: 3`

Te permite reducir el rango (en trozos) de donde aparecerán los mobs alrededor del jugador. Dependiendo del modo de juego de tu servidor y de su número de jugadores, puede que quieras reducir este valor junto con `spawn-limits` de [bukkit.yml]. Establecer este valor más bajo hará que parezca que hay más mobs a tu alrededor. Este valor debería ser menor o igual que la distancia de simulación, y nunca mayor que el rango de despawn / 16.

#### entity-activation-range

```
Buenos valores iniciales:

      animals: 16
      monsters: 24
      raiders: 48
      misc: 8
      water: 8
      villagers: 16
      flying-monsters: 48
```

Puedes establecer a qué distancia del jugador debe estar una entidad para que haga tick (hacer cosas). La reducción de estos valores ayuda al rendimiento, pero puede dar lugar a turbas irresponsables hasta que el jugador se acerca mucho a ellos. Reducir demasiado este valor puede romper ciertas granjas de mobs; las granjas de hierro son la víctima más común.

#### entity-tracking-range

```
Buenos valores iniciales:

      players: 48
      animals: 48
      monsters: 48
      misc: 32
      other: 64
```

Esta es la distancia en bloques a partir de la cual las entidades serán visibles. Simplemente no serán enviadas a los jugadores. Si se establece demasiado bajo, puede provocar que las criaturas aparezcan de la nada cerca de un jugador. En la mayoría de los casos debería ser mayor que el "rango de activación de entidades".

#### tick-inactive-villagers

`Buenos valores iniciales: false`

Esto te permite controlar si los aldeanos deben ser marcados fuera del rango de activación. Esto hará que los aldeanos procedan de forma normal e ignoren el rango de activación. Desactivar esta opción mejorará el rendimiento, pero puede resultar confuso para los jugadores en determinadas situaciones. Esto puede causar problemas con las granjas de hierro y el reabastecimiento del comercio.

#### nerf-spawner-mobs

`Buen valor inicial: true`

Puedes hacer que los monstruos generados por un generador de monstruos no tengan IA. Los monstruos nerfeados no harán nada. Puedes hacer que salten mientras están en el agua cambiando `spawner-nerfed-mobs-should-jump` a `true` en [paper-world configuration].

### [paper-world configuration]

#### despawn-ranges

```
Buenos valores iniciales:

      ambient:
        hard: 56
        soft: 30
      axolotls:
        hard: 56
        soft: 30
      creature:
        hard: 56
        soft: 30
      misc:
        hard: 56
        soft: 30
      monster:
        hard: 56
        soft: 30
      underground_water_creature:
        hard: 56
        soft: 30
      water_ambient:
        hard: 56
        soft: 30
      water_creature:
        hard: 56
        soft: 30
```

Te permite ajustar los rangos de despawn de las entidades (en bloques). Baja esos valores para eliminar más rápido a las criaturas que están lejos del jugador. Deberías mantener el rango suave alrededor de `30` y ajustar el rango duro a un poco más que tu distancia de simulación real, para que los mobs no desaparezcan inmediatamente cuando el jugador va justo más allá del punto en el que un chunk está siendo cargado (esto funciona bien debido a `delay-chunk-unloads-by` en [paper-world configuration]). Cuando un mob está fuera del rango duro, será despawneado instantáneamente. Cuando esté entre el rango suave y el duro, tendrá una probabilidad aleatoria de despawn. Tu rango duro debe ser mayor que tu rango blando. Deberías ajustar esto de acuerdo a tu distancia de visión usando `(simulation-distance * 16) + 8`. Esto tiene en cuenta parcialmente los trozos que aún no se han descargado después de que el jugador los haya visitado.

#### per-player-mob-spawns

`Buen valor inicial: true`

Esta opción decide si la aparición de mobs debe tener en cuenta cuántos mobs hay ya alrededor del jugador objetivo. Puedes evitar muchos problemas de inconsistencia en la aparición de mobs debido a que los jugadores crean granjas que ocupan todo el mobcap. Esto permitirá una experiencia de desove más parecida a la de un jugador, permitiéndote establecer `límites de desove` más bajos. La activación de esta opción tiene un ligero impacto en el rendimiento, pero queda eclipsado por las mejoras en los "límites de aparición" que permite.

#### max-entity-collisions

`Buenos valores iniciales: 2`

Sobrescribe la opción con el mismo nombre en [spigot.yml]. Permite decidir cuántas colisiones puede procesar una entidad a la vez. Un valor de `0` causará la incapacidad de empujar a otras entidades, incluyendo jugadores. Un valor de `2` debería ser suficiente en la mayoría de los casos. Vale la pena señalar que esto hará maxEntityCramming gamerule inútil si su valor es superior al valor de esta opción de configuración.

#### update-pathfinding-on-block-update

`Buen valor inicial: false`

Al desactivar esta opción, se hará menos pathfinding, lo que aumentará el rendimiento. En algunos casos, esto hará que los mobs parezcan más lentos; simplemente actualizarán pasivamente su ruta cada 5 ticks (0.25 seg).

#### fix-climbing-bypassing-cramming-rule

`Buenos valores iniciales: true`

Al activar esta opción, las entidades no se verán afectadas por el apilamiento al escalar. Esto evitará que se apilen cantidades absurdas de mobs en espacios pequeños aunque estén trepando (arañas).

#### armor-stands.tick

`Buenos valores iniciales: false`

En la mayoría de los casos se puede establecer con seguridad a `false`. Si estás usando soportes de armadura o cualquier plugin que modifique su comportamiento y experimentas problemas, vuelve a activarlo. Esto evitará que los soportes de armadura sean empujados por el agua o se vean afectados por la gravedad.

#### armor-stands.do-collision-entity-lookups

`Buen valor inicial: false`

Aquí puedes desactivar las colisiones de los soportes de armadura. Esto te ayudará si tienes muchos puestos de armadura y no necesitas que colisionen con nada.

#### tick-rates

```
Buenos valores iniciales:

  behavior:
    villager:
      validatenearbypoi: 60
      acquirepoi: 120
  sensor:
    villager:
      secondarypoisensor: 80
      nearestbedsensor: 80
      villagerbabiessensor: 40
      playersensor: 40
      nearestlivingentitysensor: 40
```

> No se recomienda cambiar estos valores por defecto mientras [DAB de Pufferfish](#dabenabled) esté activado.

Esto decide la frecuencia con la que se disparan los comportamientos y sensores especificados en ticks. `acquirepoi` para los aldeanos parece ser el comportamiento más pesado, por lo que se ha aumentado mucho. Disminuirlo en caso de problemas con los aldeanos encontrar su camino alrededor.

### [pufferfish.yml]

#### dab.enabled

`Buen valor inicial: true`

DAB (activación dinámica del cerebro) reduce la cantidad que se marca una entidad cuanto más lejos está de los jugadores. DAB funciona en un gradiente en lugar de un corte duro como EAR. En lugar de marcar completamente las entidades cercanas y apenas marcar las entidades lejanas, DAB reducirá la cantidad de marcación de una entidad basándose en el resultado de un cálculo influenciado por [dab.activation-dist-mod](#dabactivation-dist-mod).

#### dab.max-tick-freq

`Buen valor inicial: 20`

Define la cantidad más lenta a la que se moverán las entidades más alejadas de los jugadores. El aumento de este valor puede mejorar el rendimiento de las entidades lejos de la vista, pero puede romper las granjas o nerf en gran medida el comportamiento mafia. Si al activar DAB se rompen las granjas, prueba a reducir este valor.

#### dab.activation-dist-mod

`Buen valor inicial: 7`

Controla el gradiente en el que se marcan los mobs. Disminuir este valor activará DAB más cerca de los jugadores, mejorando las ganancias de rendimiento de DAB, pero afectará a cómo las entidades interactúan con su entorno y puede romper las granjas de turbas. Si al activar DAB se rompen las granjas de mobs, prueba a aumentar este valor.

#### enable-async-mob-spawning

`Buen valor inicial: true`

Si se debe habilitar el desove asíncrono de mobs. Para que esto funcione, el ajuste de Paper per-player-mob-spawns debe estar habilitado. Esta opción no genera realmente mobs de forma asíncrona, pero descarga gran parte del esfuerzo computacional involucrado en la generación de nuevos mobs a un hilo diferente. Activar esta opción no debería afectar al juego.

#### enable-suffocation-optimization

`Buen valor inicial: true`

Esta opción optimiza la comprobación de asfixia (la comprobación para ver si un monstruo está dentro de un bloque y si debería recibir daño por asfixia), limitando la tasa de comprobación al tiempo de espera del daño. Esta optimización debería ser imposible de notar, a menos que seas un jugador extremadamente técnico que esté utilizando una sincronización precisa de ticks para matar a una entidad por asfixia en el momento exacto.

#### inactive-goal-selector-throttle

`Buen valor inicial: true`

Acelera el selector de objetivos de la IA en los ticks de entidades inactivas, haciendo que las entidades inactivas actualicen su selector de objetivos cada 20 ticks en lugar de cada ticks. Puede mejorar el rendimiento en un pequeño porcentaje y tiene implicaciones menores en la jugabilidad.

### [purpur.yml]

#### zombie.aggressive-towards-villager-when-lagging

`Buen valor inicial: false`

Activar esta opción hará que los zombis dejen de apuntar a los aldeanos si el servidor está por debajo del umbral de tps establecido con `lagging-threshold` en [purpur.yml].

#### entities-can-use-portals

`Buen valor inicial: false`

Esta opción puede deshabilitar el uso del portal de todas las entidades además del jugador. Esto evita que las entidades carguen trozos cambiando mundos, lo que se gestiona en el hilo principal. Esto tiene el efecto secundario de que las entidades no pueden atravesar portales.

#### villager.lobotomize.enabled

`Buen valor inicial: true`

> Esta opción sólo debe activarse si los aldeanos causan lag. De lo contrario, las comprobaciones de búsqueda de rutas pueden reducir el rendimiento.

Los aldeanos lobotomizados son despojados de su IA y solo reponen sus ofertas cada cierto tiempo. Si activas esta opción, los aldeanos lobotomizados que no puedan encontrar su camino llegarán a su destino. Si los liberas, los deslobotomizarás.

---

## Misc

### [spigot.yml]

#### merge-radius

```
Buenos valores iniciales:

      item: 3.5
      exp: 4.0
```

Esto decide la distancia entre los items y orbes de exp a fusionar, reduciendo la cantidad de items tickeando en el suelo. Un valor demasiado alto hará que la ilusión de objetos u orbes de exp desaparezcan al fusionarse. Un valor demasiado alto romperá algunas granjas y permitirá que los objetos se teletransporten a través de los bloques. No se realizan comprobaciones para evitar que los objetos se fusionen a través de las paredes. La Exp sólo se fusiona en el momento de la creación.

#### hopper-transfer

`Buen valor inicial: 8`

Tiempo en [ticks] (https://ticks.tect.host/) que las tolvas esperarán para mover un artículo. Aumentar esto ayudará a mejorar el rendimiento si hay muchas tolvas en tu servidor, pero romperá los relojes basados en tolvas y posiblemente los sistemas de clasificación de artículos si se establece demasiado alto.

#### hopper-check

`Buen valor inicial: 8`

Tiempo en ticks entre que las tolvas comprueban si hay un objeto sobre ellas o en el inventario sobre ellas. Aumentar este valor mejorará el rendimiento si hay muchas tolvas en el servidor, pero estropeará los relojes basados en tolvas y los sistemas de clasificación de artículos basados en flujos de agua.

### [paper-world configuration]

#### alt-item-despawn-rate

```
Buenos valores de partida:

      enabled: true
      items:
        cobblestone: 300
        netherrack: 300
        sand: 300
        red_sand: 300
        gravel: 300
        dirt: 300
        grass: 300
        pumpkin: 300
        melon_slice: 300
        kelp: 300
        bamboo: 300
        sugar_cane: 300
        twisting_vines: 300
        weeping_vines: 300
        oak_leaves: 300
        spruce_leaves: 300
        birch_leaves: 300
        jungle_leaves: 300
        acacia_leaves: 300
        dark_oak_leaves: 300
        mangrove_leaves: 300
        cactus: 300
        diorite: 300
        granite: 300
        andesite: 300
        scaffolding: 600
```

Esta lista te permite establecer un tiempo alternativo (en ticks) para que ciertos tipos de objetos caídos desaparezcan más rápido o más lento que por defecto. Esta opción se puede utilizar en lugar de los plugins de limpieza de elementos junto con `merge-radius` para mejorar el rendimiento.

#### redstone-implementation

`Buen valor inicial: ALTERNATE_CURRENT`

Sustituye el sistema de redstone por versiones más rápidas y alternativas que reducen las actualizaciones redundantes de bloques, reduciendo la cantidad de lógica que tiene que calcular tu servidor. El uso de una implementación no vainilla puede introducir inconsistencias menores con redstone muy técnico, pero las ganancias de rendimiento superan con creces los posibles problemas de nicho. Una opción de implementación no vainilla puede solucionar además otras incoherencias de redstone causadas por CraftBukkit.

La implementación de `ALTERNATE_CURRENT` se basa en el mod [Alternate Current](https://modrinth.com/mod/alternate-current). Puedes encontrar más información sobre este algoritmo en su página de recursos.

#### hopper.disable-move-event

`Buen valor inicial: false`

El evento `InventoryMoveItemEvent` no se dispara a menos que haya un plugin escuchando activamente ese evento. Esto significa que sólo debes ponerlo a true si tienes tal(es) plugin(s) y no te importa que no puedan actuar sobre este evento. **No lo establezca a true si quiere usar plugins que escuchen este evento, por ejemplo, plugins de protección.

#### hopper.ignore-occluding-blocks

`Buen valor inicial: true`

Determina si las tolvas ignorarán los contenedores dentro de bloques llenos, por ejemplo la tolva minecart dentro de un bloque de arena o grava. Si se mantiene activada esta opción, se romperán algunos artilugios que dependen de ese comportamiento.

#### tick-rates.mob-spawner

`Buen valor inicial: 2`

Esta opción te permite configurar la frecuencia con la que deben marcarse los generadores. Valores más altos significan menos lag si tienes muchos spawners, aunque si se establece demasiado alto (en relación con el retraso de tus spawners) las tasas de spawn de mobs disminuirán.

#### optimize-explosions

`Buen valor inicial: true`

Establecer esto a `true` reemplaza el algoritmo de explosión vainilla por uno más rápido, a costa de una ligera inexactitud al calcular el daño de la explosión. Esto no suele ser perceptible.

#### treasure-maps.enabled

`Buen valor inicial: false`

Generar mapas del tesoro es extremadamente caro y puede colgar un servidor si la estructura que está intentando localizar está en un trozo no generado. Sólo es seguro activar esto si has pregenerado tu mundo y establecido un borde de mundo vainilla.

#### treasure-maps.find-already-discovered

```
Buenos valores de partida:
      loot-tables: true
      villager-trade: true
```

El valor por defecto de esta opción obliga a los mapas recién generados a buscar estructuras inexploradas, que suelen estar en trozos aún no generados. Establecer esto a `true` hace que los mapas puedan llevar a las estructuras que fueron descubiertas anteriormente. Si no cambias esto a `true` puedes experimentar que el servidor se cuelgue o se cuelgue al generar nuevos mapas del tesoro. villager-trade" es para mapas comerciados por aldeanos y "loot-tables" se refiere a cualquier cosa que genere botín dinámicamente como cofres del tesoro, cofres de mazmorras, etc.

#### tick-rates.grass-spread

`Buen valor inicial: 4`

Tiempo en ticks entre que el servidor intenta esparcir hierba o micelio. Esto hará que las grandes áreas de tierra tarden un poco más en convertirse en hierba o micelio. Establecerlo en torno a `4` debería funcionar bien si quieres reducirlo sin que se note la disminución de la velocidad de propagación.

#### tick-rates.container-update

`Buen valor inicial: 1`

Tiempo en ticks entre actualizaciones de contenedores. Aumentarlo puede ayudar si las actualizaciones de los contenedores te causan problemas (rara vez ocurre), pero facilita que los jugadores experimenten desincronización al interactuar con los inventarios (objetos fantasma).

#### non-player-arrow-despawn-rate

`Buen valor inicial: 20`

Tiempo en ticks después del cual las flechas disparadas por los mobs deberían desaparecer después de golpear algo. De todas formas, los jugadores no pueden recogerlas, así que también puedes establecerlo en algo como `20` (1 segundo).

#### creative-arrow-despawn-rate

`Buen valor inicial: 20`

Tiempo en ticks tras el cual las flechas disparadas por los jugadores en modo creativo deberían desaparecer después de golpear algo. De todas formas, los jugadores no pueden recogerlas, así que también puedes establecerlo en algo como `20` (1 segundo).

### [pufferfish.yml]

#### disable-method-profiler

`Buen valor inicial: true`

Esta opción desactivará algunos perfiles adicionales realizados por el juego. Este perfilado no es necesario para funcionar en producción y puede causar retrasos adicionales..

### [purpur.yml]

#### dolphin.disable-treasure-searching

`Buen valor inicial: true`

Evita que los delfines realicen búsquedas de estructuras similares a los mapas del tesoro

#### teleport-if-outside-border

`Buen valor inicial: true`

Te permite teletransportar al jugador al spawn del mundo si se encuentra fuera de la frontera del mundo. Resulta útil, ya que la frontera del mundo de vainilla se puede evitar y el daño que causa al jugador se puede mitigar.

---

## Helpers

### [paper-world configuration]

#### anti-xray.enabled

`Buen valor inicial: true`

Activa esta opción para ocultar los minerales de los rayos X. Para una configuración detallada de esta función, consulte [Configuring Anti-Xray](https://docs.papermc.io/paper/anti-xray). Activar esta opción disminuirá el rendimiento, sin embargo es mucho más eficiente que cualquier plugin anti-xray. En la mayoría de los casos el impacto en el rendimiento será insignificante.

#### nether-ceiling-void-damage-height

`Buen valor inicial: 127`

Si esta opción es mayor que `0`, los jugadores por encima del nivel y establecido sufrirán daños como si estuvieran en el vacío. Esto evitará que los jugadores utilicen el techo del Nether. El nether vainilla tiene una altura de 128 bloques, por lo que probablemente deberías establecerlo en `127`. Si modificas la altura del nether de alguna manera, deberías establecerla en `[your_nether_height] - 1`.

---

# Java startup flags
[Vanilla Minecraft y Minecraft software de servidor en la versión 1.19 requiere Java 17 o superior](https://docs.papermc.io/java-install-update). Oracle ha cambiado su concesión de licencias, y ya no hay una razón de peso para obtener su Java de ellos. Los proveedores recomendados son [Adoptium](https://adoptium.net/) y [Amazon Corretto](https://aws.amazon.com/corretto/). Implementaciones alternativas de JVM como OpenJ9 o GraalVM pueden funcionar, sin embargo no están soportadas por Paper y se sabe que causan problemas, por lo que no se recomiendan actualmente.

Tu recolector de basura puede ser configurado para reducir los picos de lag causados por grandes tareas del recolector de basura. Puedes encontrar banderas de inicio optimizadas para servidores Minecraft [aquí](https://docs.papermc.io/paper/aikars-flags) [`SOG`]. Ten en cuenta que esta recomendación no funcionará en implementaciones JVM alternativas.
Se recomienda utilizar el generador de banderas de inicio [flags.sh](https://docs.papermc.io/paper/aikars-flags) para obtener las banderas de inicio correctas para tu servidor.

Además, añadir la bandera beta `--add-modules=jdk.incubator.vector` antes de `-jar` en tus banderas de inicio puede mejorar el rendimiento. Esta bandera permite a Pufferfish utilizar instrucciones SIMD en tu CPU, haciendo algunas matemáticas más rápidas. Actualmente, sólo se utiliza para hacer el renderizado en mapas de plugins de juegos (como imageonmaps) posiblemente 8 veces más rápido.

# Plugins "demasiado bueno para ser verdad"

## Plugins que eliminan items del suelo
Absolutamente innecesarios ya que pueden ser reemplazados con [merge-radius](#merge-radius) y [alt-item-despawn-rate](#alt-item-despawn-rate) y francamente, son menos configurables que las configuraciones básicas del servidor. Tienden a utilizar más recursos escaneando y eliminando elementos que no eliminándolos en absoluto.

## Plugins Mob stacker
Es realmente difícil justificar el uso de uno. Apilar entidades generadas de forma natural causa más lag que no apilarlas debido a que el servidor está constantemente intentando generar más mobs. El único caso de uso "aceptable" es para los spawners en servidores con una gran cantidad de spawners.

## Plugins que activan/desactivan otros plugins
Cualquier cosa que habilite o deshabilite plugins en tiempo de ejecución es extremadamente peligrosa. Cargar un plugin así puede causar errores fatales con los datos de rastreo y deshabilitar un plugin puede llevar a errores debido a la eliminación de la dependencia. El comando `/reload` sufre exactamente los mismos problemas y puedes leer más sobre ellos en [me4502's blog post](https://madelinemiller.dev/blog/problem-with-reload/)

# ¿Qué se retrasa? - medir el rendimiento

## mspt
Paper ofrece un comando `/mspt` que te dirá cuánto tiempo ha tardado el servidor en calcular los ticks recientes. Si el primer y segundo valor que ves son inferiores a 50, ¡enhorabuena! Su servidor no se está retrasando. Si el tercer valor es superior a 50, significa que ha habido al menos un tick que ha tardado más. Esto es completamente normal y ocurre de vez en cuando, así que no te asustes.
  
## Spark
[Spark](https://spark.lucko.me/) es un plugin que te permite perfilar el uso de CPU y memoria de tu servidor. Puedes leer cómo usarlo [en su wiki](https://spark.lucko.me/docs/). También hay una guía sobre cómo encontrar la causa de los picos de lag [aquí](https://spark.lucko.me/docs/guides/Finding-lag-spikes).

## Timings
Una forma de ver lo que puede estar pasando cuando tu servidor se está retrasando es Timings. Timings es una herramienta que le permite ver exactamente qué tareas están tomando más tiempo. Es la herramienta más básica de solución de problemas y si usted pide ayuda con respecto a lag lo más probable es que se le preguntó por su Timings. Timings es conocido por tener un serio impacto en el rendimiento de los servidores, se recomienda utilizar el plugin Spark sobre Timings y utilizar Purpur o Pufferfish para desactivar Timings.

Para obtener Timings de tu servidor, sólo tienes que ejecutar el comando `/timings paste` y hacer clic en el enlace que se te proporciona. Puedes compartir este enlace con otras personas para que te ayuden. También es fácil equivocarse si no sabes lo que estás haciendo. Hay un [videotutorial de Aikar](https://www.youtube.com/watch?v=T4J0A9l7bfQ) detallado sobre cómo leerlos.

[`SOG`]: https://www.spigotmc.org/threads/guide-server-optimization%E2%9A%A1.283181/
[server.properties]: https://minecraft.fandom.com/wiki/Server.properties
[bukkit.yml]: https://bukkit.fandom.com/wiki/Bukkit.yml
[spigot.yml]: https://www.spigotmc.org/wiki/spigot-configuration/
[paper-global configuration]: https://docs.papermc.io/paper/reference/global-configuration
[paper-world configuration]: https://docs.papermc.io/paper/reference/world-configuration
[purpur.yml]: https://purpurmc.org/docs/Configuration/
[pufferfish.yml]: https://docs.pufferfish.host/setup/pufferfish-fork-configuration/
[Petal]: https://github.com/Bloom-host/Petal
[Translation]: https://tect.host/
