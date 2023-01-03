# Errores comunes y mejores prácticas

El objetivo de este artículo es explicar las dificultades más comunes a las que se enfrentan los propietarios de servidores.

## Hacer siempre copias de seguridad
Hay dos tipos de personas: las que hacen copias de seguridad y las que empezarán a hacerlas. Es sólo cuestión de tiempo que se produzca una pérdida de datos. Haz siempre copias para evitar perder tus mundos o datos de plugins. Puedes aplicar esto a cualquier flujo de trabajo relacionado con la informática, no sólo a minecraft.

## No utilices software obsoleto
Al ejecutar versiones de software no actualizadas te arriesgas a que los jugadores abusen de exploits no parcheados, incluyendo la duplicación de ítems (ítems infinitos). También añade un factor de inconveniencia, ya que tus jugadores tienen que bajar específicamente la versión de su cliente para que coincida con tu servidor. Esto puede evitarse utilizando un hack de protocolo, pero no es lo ideal.

## No ejecutes más Bukkit/Spigot
Bukkit y Spigot están básicamente en modo de mantenimiento. Se actualizan cada vez que hay una nueva versión y si se encuentra un exploit crítico, pero no añaden ninguna actualización de rendimiento. Esto significa que cualquier problema de rendimiento que puedas experimentar en esos softwares nunca mejorará con el tiempo. Para evitarlo, actualice a [Paper](https://papermc.io/downloads) o [Purpur](https://purpurmc.org/downloads). Los plugins Bukkit/Spigot funcionarán igual de bien (quizás incluso mejor) con el software de servidor mencionado. Si no lo hacen, entonces es seguro asumir que el desarrollador del plugin está haciendo cosas que no debería o hizo un trabajo negligente al crear su plugin. También añaden parches de optimización como un sistema de carga de trozos que puede aprovechar múltiples hilos de cpu o un ajuste que permite al servidor marcar menos trozos de los que realmente envía al jugador. Ver la [guía principal de optimización](https://github.com/YouHaveTrouble/minecraft-optimization) para más detalles.

## Evita el alojamiento compartido si es posible
Los alojamientos compartidos suelen ser la opción más barata, y eso es por una razón válida. Te ofrecen 2 tipos de recursos - garantizados y compartidos. Los recursos garantizados suelen ser irrisoriamente bajos y pueden no ser suficientes para hacer funcionar un servidor para unos pocos jugadores. Por otro lado, los recursos compartidos suelen ser suficientes para hacer funcionar un servidor con un rendimiento decente. Los recursos compartidos, como su nombre indica, se comparten entre tu servidor y otros servidores de la misma máquina física. Tu servidor sólo puede beneficiarse de tenerlos cuando ningún otro servidor los utiliza. La situación en la que tu servidor utiliza totalmente los recursos compartidos es prácticamente imposible, ya que la mayoría de los alojamientos compartidos sobrevenden sus recursos. Al igual que los billetes de avión, el sitio de alojamiento vende más recursos de los que tiene disponibles con la esperanza de que no se utilicen todos. Esto a menudo conduce a situaciones en las que todos los servidores están atascados porque no hay suficientes recursos de sobra.

## Evite paquetes de datos que utilicen funciones de comandos
Los paquetes de datos que ejecutan comandos son extremadamente lentos. Puede que no sea mucho con pocos jugadores, pero eso no se escala bien con el número de jugadores y hará que tu servidor se ralentice rápidamente a medida que ganes jugadores. Los paquetes de datos que modifican biomas, tablas de botín, etc. están bien. Es mejor que busques un plugin alternativo.

## Elección del hardware
No te guíes sólo por la cantidad de RAM que necesitas. Deberías centrarte en qué tipo de CPU deberías usar, ya que la CPU es la parte más importante del servidor. Usted quiere algo que [se clasifica bien en el rendimiento de un solo núcleo](https://www.cpubenchmark.net/singleThread.html), como un servidor se ejecuta principalmente en un hilo. Sin embargo, los hilos múltiples se utilizan desde hace bastante tiempo en sistemas como async chunk loading on paper.

Deberías evitar absolutamente los discos duros (HDDs). Sus velocidades son simplemente demasiado lento para justificar la ejecución de un servidor en ellos desde Minecraft es pesado en las operaciones de E / S (especialmente con altas distancias de vista y mayor número de jugadores). Una unidad de estado sólido (SSD) es una opción mucho mejor debido a que es mucho más rápido de E / S.
