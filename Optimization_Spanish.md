<div align="center">

<img src="https://raw.githubusercontent.com/zMarkitos/zMOptimization_Minecraft_2026/refs/heads/main/assets/banner.svg" alt="Minecraft Optimization Guide" width="100%">

# Guia de Optimizacion de Servidores Minecraft
### La guia mas completa y actualizada para 2026

[![Version](https://img.shields.io/badge/Version-2026.1-brightgreen?style=for-the-badge)](https://github.com)
[![Minecraft](https://img.shields.io/badge/Minecraft-1.20.x--1.21.x-blue?style=for-the-badge)](https://minecraft.net)
[![Paper](https://img.shields.io/badge/Paper-Compatible-orange?style=for-the-badge)](https://papermc.io)
[![Java](https://img.shields.io/badge/Java-21+-red?style=for-the-badge)](https://adoptium.net)
[![Licencia](https://img.shields.io/badge/Licencia-MIT-purple?style=for-the-badge)](LICENSE)
[![ES](https://img.shields.io/badge/Lang-Spanish-blue?style=for-the-badge)](README.en.md)

<br>

> **El rendimiento de tu servidor no depende solo del hardware.**
> La configuracion correcta puede ser la diferencia entre 20 TPS estables y un servidor que colapsa con 10 jugadores.

<br>

[Introduccion](#introduccion) •
[Preparativos](#preparativos) •
[Configuracion](#configuraciones) •
[Java Flags](#java-startup-flags) •
[Plugins](#plugins-mitos-y-realidades) •
[Diagnostico](#midiendo-el-rendimiento) •
[Exploits](#exploits-y-como-solucionarlos)

</div>

---

## Tabla de Contenidos

- [Introduccion](#introduccion)
- [Preparativos](#preparativos)
  - [Eleccion del JAR de servidor](#eleccion-del-jar-de-servidor)
  - [Pre-generacion del mapa](#pre-generacion-del-mapa)
  - [Redes y Proxies](#redes-y-proxies)
- [Configuraciones](#configuraciones)
  - [Networking](#networking)
  - [Chunks](#chunks)
  - [Mobs](#mobs)
  - [Miscelaneos](#miscelaneos)
  - [Helpers y Seguridad](#helpers-y-seguridad)
- [Java Startup Flags](#java-startup-flags)
  - [G1GC Recomendado](#g1gc-recomendado-para-la-mayoria)
  - [ZGC Avanzado](#zgc-para-servidores-de-alto-trafico)
- [Plugins Mitos y Realidades](#plugins-mitos-y-realidades)
- [Midiendo el Rendimiento](#midiendo-el-rendimiento)
- [Exploits y Como Solucionarlos](#exploits-y-como-solucionarlos)
- [FAQ](#faq)
- [Contribuir](#contribuir)
- [Fuentes](#fuentes)

---

## Introduccion

> **Nota importante para usuarios de Vanilla, Fabric o Spigot:**
> Ve a `server.properties` y cambia `sync-chunk-writes` a `false`. En Paper y sus forks esto esta forzado a `false`, pero en otras implementaciones debes cambiarlo manualmente. Esto permite guardar chunks fuera del hilo principal, reduciendo drasticamente la carga del bucle principal.

**Nunca existira una configuracion perfecta universal.** Cada servidor tiene sus propias necesidades y limites. Esta guia pretende ayudarte a entender que opciones tienen impacto real en el rendimiento y que cambian exactamente, para que puedas tomar decisiones informadas.

### Que aprenderas

| Categoria | Beneficio Esperado |
|-----------|-------------------|
| Software correcto | Hasta 3x mas rendimiento vs Vanilla/Spigot |
| Configuracion de chunks | Reduccion del 40-60% en uso de CPU |
| Optimizacion de mobs | Mejora de 2-5 TPS en servidores cargados |
| Java Flags | Eliminacion del 80-90% de lag spikes por GC |
| Plugins correctos | Evitar perdidas de hasta 5 TPS por plugins malos |

---

## Preparativos

### Eleccion del JAR de Servidor

La eleccion del software es **la decision mas importante**. Aqui el panorama completo en 2026:

#### Software Recomendado

```
Vanilla > Spigot > Paper > Pufferfish > Purpur
         (cada fork hereda las optimizaciones del anterior)
```

| Software | Descripcion | Para quien | Enlace |
|----------|-------------|------------|--------|
| **Paper** | El fork mas popular. Corrige inconsistencias de gameplay y mejora el rendimiento significativamente. Base de toda la jerarquia. | La mayoria de servidores | [papermc.io](https://papermc.io) |
| **Pufferfish** | Fork de Paper enfocado en rendimiento puro. Incluye DAB (Dynamic Activation Brain), spawn asincrono de mobs, SIMD para mapas y mas. | Servidores con 50+ jugadores o granjas pesadas | [pufferfish.host](https://pufferfish.host) |
| **Purpur** | Fork de Pufferfish. Incluye todas las optimizaciones mas cientos de opciones de personalizacion del gameplay. Las funciones extra estan desactivadas por defecto. | Servidores que necesitan personalizacion extrema | [purpurmc.org](https://purpurmc.org) |
| **Folia** | Fork experimental de Paper con multithreading por regiones. Rompe muchos plugins pero puede manejar cargas masivas. | Servidores con 150+ jugadores, uso avanzado | [papermc.io/folia](https://papermc.io/software/folia) |

> **Recomendacion 2026:** Para un SMP publico o servidor de supervivencia con plugins, usa **Purpur** o **Pufferfish**. Para una red grande, considera **Folia** solo si tus plugins lo soportan.

#### Software a Evitar

- **Bukkit / CraftBukkit / Spigot** — Extremadamente anticuados en rendimiento. No hay razon para usarlos en 2026.
- **JARs de pago que prometen async todo** — 99.99% son estafas o implementaciones rotas.
- **Forks exoticos downstream de Purpur** — Casi siempre tienen problemas de estabilidad sin beneficio real de rendimiento.
- **Versiones inestables/snapshot** — Si quieres rendimiento, usa releases estables.

---

### Pre-generacion del Mapa

La pre-generacion en 2026 ya no es estrictamente necesaria en CPUs modernas, pero sigue siendo util si:

- Usas plugins de mapas del mundo como [Pl3xMap](https://github.com/jpenilla/Pl3xMap) o [Dynmap](https://github.com/webbukkit/dynmap)
- Tienes una CPU muy limitada de un solo hilo
- Quieres eliminar completamente el lag de generacion de chunks nuevos

**Plugin recomendado:** [Chunky](https://modrinth.com/plugin/chunky)

```
IMPORTANTE: Configura siempre un borde de mundo ANTES de pregenerar
/worldborder set [diametro]
```

> **El Overworld, Nether y End tienen bordes de mundo separados.** El Nether es 1/8 del tamano del Overworld. Configura cada dimension correctamente o los jugadores pueden terminar fuera del borde.

> **El borde de mundo vanilla tambien limita el rango de busqueda de mapas del tesoro**, que puede causar picos de lag masivos si no esta configurado.

---

### Redes y Proxies

Si administras una **red de servidores**, la eleccion del proxy es crucial:

| Proxy | Estado en 2026 | Recomendacion |
|-------|---------------|---------------|
| **BungeeCord** | Funcional pero legacy | Solo si necesitas plugins especificos sin alternativa |
| **Waterfall** | Fork de BungeeCord, mas estable | Alternativa si migas desde BungeeCord |
| **Velocity** | Moderno, alto rendimiento, activamente desarrollado | RECOMENDADO para cualquier red nueva |

**Velocity**, desarrollado por el equipo de PaperMC, es la unica opcion real para redes nuevas en 2026. Soporta 1000+ jugadores con degradacion minima de rendimiento, tiene seguridad mejorada y una API moderna.

---

## Configuraciones

> **Consejo:** Realiza cambios de a uno y mide el impacto con [Spark](#spark) antes de continuar.

---

### Networking

#### `server.properties`

<details>
<summary><b>network-compression-threshold</b> — Valor recomendado: 256</summary>

```properties
network-compression-threshold=256
```

Define el tamano minimo de un paquete antes de que el servidor lo comprima.

- **Mas alto** — Menos CPU, mas ancho de banda
- **`-1`** — Desactiva la compresion (ideal si el proxy esta en la misma maquina con menos de 2ms de ping)
- **Mas bajo** — Mas CPU, menos ancho de banda (perjudica conexiones lentas)

</details>

#### `purpur.yml`

<details>
<summary><b>use-alternate-keepalive</b> — Valor recomendado: true</summary>

```yaml
settings:
  use-alternate-keepalive: true
```

Envia un paquete keepalive por segundo al jugador. Solo expulsa al jugador si ninguno fue respondido en 30 segundos. Reduce desconexiones en jugadores con conexiones inestables.

> Incompatible con **TCPShield**.

</details>

---

### Chunks

#### `server.properties`

<details>
<summary><b>simulation-distance</b> — Valor recomendado: 4</summary>

```properties
simulation-distance=4
```

La distancia (en chunks) alrededor del jugador donde las cosas realmente suceden: hornos, cultivos, arboles, mobs, etc.

Mantenerlo bajo (3-4) es intencional. Se complementa con `view-distance` para que los jugadores vean mas lejos sin el mismo impacto en rendimiento.

</details>

<details>
<summary><b>view-distance</b> — Valor recomendado: 7</summary>

```properties
view-distance=7
```

Chunks enviados visualmente al cliente. La distancia total sera el mayor valor entre `simulation-distance` y `view-distance`.

> Ejemplo: simulation=4, view=7 — el cliente ve 7 chunks, pero solo 4 estan simulados activamente.

</details>

#### `paper-world.yml`

<details>
<summary><b>delay-chunk-unloads-by</b> — Valor recomendado: 10s</summary>

```yaml
chunks:
  delay-chunk-unloads-by: 10s
```

Mantiene los chunks cargados 10 segundos despues de que un jugador se aleje. Evita cargar/descargar constantemente los mismos chunks cuando un jugador va y viene.

</details>

<details>
<summary><b>max-auto-save-chunks-per-tick</b> — Valor recomendado: 8</summary>

```yaml
chunks:
  max-auto-save-chunks-per-tick: 8
```

Distribuye el guardado de mundo en el tiempo para mejor rendimiento promedio. Con 20-30+ jugadores, considera aumentarlo a `12` o `16`.

</details>

<details>
<summary><b>prevent-moving-into-unloaded-chunks</b> — Valor recomendado: true</summary>

```yaml
chunks:
  prevent-moving-into-unloaded-chunks: true
```

Evita que los jugadores entren en chunks no cargados, previniendo cargas de sincronizacion que atascan el hilo principal.

</details>

<details>
<summary><b>entity-per-chunk-save-limit</b> — Ver valores recomendados</summary>

```yaml
chunks:
  entity-per-chunk-save-limit:
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

Limita cuantas entidades de cada tipo pueden guardarse por chunk. Previene crashes al cargar chunks con cantidades absurdas de proyectiles guardados.

</details>

#### `pufferfish.yml`

<details>
<summary><b>max-loads-per-projectile</b> — Valor recomendado: 8</summary>

```yaml
max-loads-per-projectile: 8
```

Limita cuantos chunks puede cargar un proyectil durante su trayectoria. Reduce el lag de enderpearls y tridentes.

</details>

---

### Mobs

#### `bukkit.yml`

<details>
<summary><b>spawn-limits</b> — Ver valores recomendados</summary>

```yaml
spawn-limits:
  monsters: 20
  animals: 5
  water-animals: 2
  water-ambient: 2
  water-underground-creature: 3
  axolotls: 3
  ambient: 1
```

La formula es: `[jugadores activos] x [limite]`. A menor numero, menos trabajo para el servidor. Combinar con `per-player-mob-spawns: true` en Paper para distribucion equitativa.

</details>

<details>
<summary><b>ticks-per</b> — Ver valores recomendados</summary>

```yaml
ticks-per:
  monster-spawns: 10
  animal-spawns: 400
  water-spawns: 400
  water-ambient-spawns: 400
  water-underground-creature-spawns: 400
  axolotl-spawns: 400
  ambient-spawns: 400
```

Frecuencia (en ticks) con la que el servidor intenta generar cada tipo de entidad. Los animales y mobs de agua no necesitan aparecer cada tick, ya que no mueren tan rapido.

</details>

#### `spigot.yml`

<details>
<summary><b>mob-spawn-range</b> — Valor recomendado: 3</summary>

```yaml
mob-spawn-range: 3
```

Rango en chunks donde pueden aparecer mobs alrededor del jugador. Debe ser menor o igual a simulation-distance y nunca mayor que el rango de despawn dividido entre 16.

</details>

<details>
<summary><b>entity-activation-range</b> — Ver valores recomendados</summary>

```yaml
entity-activation-range:
  animals: 16
  monsters: 24
  raiders: 48
  misc: 8
  water: 8
  villagers: 16
  flying-monsters: 48
```

Distancia desde el jugador a la que una entidad hace tick. Reducirlo mejora el rendimiento pero puede afectar granjas de mobs.

</details>

<details>
<summary><b>entity-tracking-range</b> — Ver valores recomendados</summary>

```yaml
entity-tracking-range:
  players: 48
  animals: 48
  monsters: 48
  misc: 32
  other: 64
```

Distancia en bloques a la que las entidades son visibles para los clientes. Debe ser mayor que `entity-activation-range`.

</details>

<details>
<summary><b>nerf-spawner-mobs</b> — Valor recomendado: true</summary>

```yaml
nerf-spawner-mobs: true
```

Los mobs generados por spawners no tendran IA. Mejora drasticamente el rendimiento en servidores con muchos spawners. Los mobs pueden saltar en el agua si `spawner-nerfed-mobs-should-jump: true` en paper-world.

</details>

#### `paper-world.yml`

<details>
<summary><b>despawn-ranges</b> — Ver valores recomendados</summary>

```yaml
entities:
  spawning:
    despawn-ranges:
      ambient:
        hard: 72
        soft: 30
      creature:
        hard: 72
        soft: 30
      monster:
        hard: 72
        soft: 30
      water_creature:
        hard: 72
        soft: 30
```

- **Soft range:** probabilidad aleatoria de despawn
- **Hard range:** despawn instantaneo
- Formula recomendada: `(simulation-distance x 16) + 8`

</details>

<details>
<summary><b>per-player-mob-spawns</b> — Valor recomendado: true</summary>

```yaml
entities:
  spawning:
    per-player-mob-spawns: true
```

Distribuye el spawn de mobs equitativamente entre jugadores. Evita que un jugador con una granja enorme acapare toda la mob cap.

</details>

<details>
<summary><b>max-entity-collisions</b> — Valor recomendado: 2</summary>

```yaml
entities:
  max-entity-collisions: 2
```

Limita cuantas colisiones puede procesar una entidad por tick. `0` hace imposible empujar entidades. `2` es suficiente para la mayoria de casos.

</details>

<details>
<summary><b>update-pathfinding-on-block-update</b> — Valor recomendado: false</summary>

```yaml
entities:
  update-pathfinding-on-block-update: false
```

Reduce el pathfinding innecesario. Los mobs actualizaran su ruta pasivamente cada 5 ticks en lugar de en cada actualizacion de bloque.

</details>

#### `pufferfish.yml` — DAB (Dynamic Activation Brain)

> **Una de las optimizaciones mas importantes de Pufferfish.**

<details>
<summary><b>dab.enabled</b> — Valor recomendado: true</summary>

```yaml
dab:
  enabled: true
  max-tick-freq: 20
  activation-dist-mod: 7
```

DAB reduce cuanto "piensa" una entidad segun que tan lejos esta de los jugadores. A diferencia de EAR (que hace un corte brusco), DAB trabaja en gradiente:

- **Entidades cercanas** — tick completo
- **Entidades lejanas** — tick reducido progresivamente

| Opcion | Descripcion | Recomendado |
|--------|-------------|-------------|
| `max-tick-freq` | Velocidad mas lenta para entidades lejanas | `20` |
| `activation-dist-mod` | Que tan cerca del jugador activa DAB | `7` |

> Si DAB rompe tus granjas, prueba aumentando `activation-dist-mod` o reduciendo `max-tick-freq`.

</details>

<details>
<summary><b>enable-async-mob-spawning</b> — Valor recomendado: true</summary>

```yaml
enable-async-mob-spawning: true
```

Mueve el calculo computacional del spawn de mobs a un hilo separado. Requiere `per-player-mob-spawns: true` en Paper. No genera mobs de forma asincrona, solo optimiza el calculo previo.

</details>

#### `purpur.yml`

<details>
<summary><b>villager.lobotomize.enabled</b> — Valor recomendado: true si hay lag de aldeanos</summary>

```yaml
mobs:
  villager:
    lobotomize:
      enabled: true
      check-interval: 100
```

Los aldeanos que no pueden seguir una ruta pierden su IA y solo reabastecen ofertas periodicamente. Activa solo si los aldeanos causan lag.

</details>

<details>
<summary><b>entities-can-use-portals</b> — Valor recomendado: false</summary>

```yaml
settings:
  entities-can-use-portals: false
```

Previene que entidades (no jugadores) usen portales y carguen chunks al cambiar dimensiones.

</details>

---

### Miscelaneos

#### `spigot.yml`

<details>
<summary><b>merge-radius</b> — Ver valores recomendados</summary>

```yaml
merge-radius:
  item: 3.5
  exp: 4.0
```

Distancia a la que items y orbes de exp se fusionan entre si, reduciendo entidades en el suelo. Muy alto puede hacer que items "teletransporten" a traves de paredes.

</details>

<details>
<summary><b>hopper-transfer / hopper-check</b> — Valor recomendado: 8</summary>

```yaml
hopper-transfer: 8
hopper-check: 8
```

Ticks de espera entre acciones de tolvas. Mejora significativamente el rendimiento con muchas tolvas. Valores muy altos pueden romper relojes de tolvas.

</details>

#### `paper-world.yml`

<details>
<summary><b>alt-item-despawn-rate</b> — Ver valores recomendados</summary>

```yaml
entities:
  spawning:
    alt-item-despawn-rate:
      enabled: true
      items:
        cobblestone: 300
        netherrack: 300
        sand: 300
        red_sand: 300
        gravel: 300
        dirt: 300
        short_grass: 300
        pumpkin: 300
        melon_slice: 300
        kelp: 300
        bamboo: 300
        sugar_cane: 300
        oak_leaves: 300
        spruce_leaves: 300
        cactus: 300
        diorite: 300
        granite: 300
        andesite: 300
        scaffolding: 600
```

Items comunes y sin valor desaparecen mas rapido, reduciendo entidades en el suelo. Alternativa eficiente a plugins de limpieza.

</details>

<details>
<summary><b>redstone-implementation</b> — Valor recomendado: ALTERNATE_CURRENT</summary>

```yaml
redstone:
  implementation: ALTERNATE_CURRENT
```

Reemplaza el sistema de redstone vanilla con una implementacion mas rapida basada en el mod [Alternate Current](https://modrinth.com/mod/alternate-current). Reduce actualizaciones redundantes de bloques.

> Puede introducir inconsistencias menores con redstone muy tecnica, pero la ganancia de rendimiento supera ampliamente los posibles problemas.

</details>

<details>
<summary><b>optimize-explosions</b> — Valor recomendado: true</summary>

```yaml
misc:
  optimize-explosions: true
```

Reemplaza el algoritmo de explosion vanilla por uno mas rapido con minima diferencia de calculo de dano.

</details>

<details>
<summary><b>treasure-maps.enabled</b> — Valor recomendado: false</summary>

```yaml
environment:
  treasure-maps:
    enabled: false
    find-already-discovered:
      loot-tables: true
      villager-trade: true
```

Generar mapas del tesoro es extremadamente caro y puede colgar el servidor si la estructura esta en chunks no generados. Solo activa si has pregenerado el mundo completamente.

</details>

<details>
<summary><b>non-player-arrow-despawn-rate / creative-arrow-despawn-rate</b> — Valor recomendado: 20</summary>

```yaml
entities:
  spawning:
    non-player-arrow-despawn-rate: 20
    creative-arrow-despawn-rate: 20
```

Las flechas de mobs y modo creativo desaparecen en 1 segundo (20 ticks) tras impactar. Los jugadores no pueden recogerlas de todas formas.

</details>

---

### Helpers y Seguridad

#### `paper-world.yml`

<details>
<summary><b>anti-xray.enabled</b> — Valor recomendado: true</summary>

```yaml
anticheat:
  anti-xray:
    enabled: true
    engine-mode: 2
    hidden-blocks:
      - copper_ore
      - deepslate_copper_ore
      - gold_ore
      - deepslate_gold_ore
      - iron_ore
      - deepslate_iron_ore
      - coal_ore
      - deepslate_coal_ore
      - lapis_ore
      - deepslate_lapis_ore
      - mossy_cobblestone
      - obsidian
      - chest
      - diamond_ore
      - deepslate_diamond_ore
      - redstone_ore
      - deepslate_redstone_ore
      - clay
      - emerald_ore
      - deepslate_emerald_ore
      - ender_chest
```

Mucho mas eficiente que cualquier plugin anti-xray. El impacto en rendimiento es minimo.

- **Mode 1:** Oculta ores especificos
- **Mode 2 (Recomendado):** Reemplaza bloques con falsos ores, mas seguro pero mas uso de CPU

</details>

<details>
<summary><b>nether-ceiling-void-damage-height</b> — Valor recomendado: 127</summary>

```yaml
environment:
  nether-ceiling-void-damage-height: 127
```

Dano de vacio sobre el techo del Nether (altura 128). Evita que los jugadores exploten el techo del Nether. Si modificas la altura del Nether: `[tu_altura_nether] - 1`.

</details>

---

## Java Startup Flags

> **Minecraft 1.20.4+ requiere Java 21.**
> Proveedores recomendados: [Adoptium](https://adoptium.net/) y [Amazon Corretto](https://aws.amazon.com/corretto/)
> No uses Oracle JDK — cambios de licencia en 2023 lo hacen inadecuado para produccion.

### G1GC Recomendado para la mayoria

G1GC sigue siendo la mejor opcion para la mayoria de servidores en 2026. Las flags de Aikar son el estandar de la industria:

```bash
java -Xms8G -Xmx8G -XX:+UseG1GC -XX:+ParallelRefProcEnabled -XX:MaxGCPauseMillis=200 -XX:+UnlockExperimentalVMOptions -XX:+DisableExplicitGC -XX:+AlwaysPreTouch -XX:G1NewSizePercent=30 -XX:G1MaxNewSizePercent=40 -XX:G1HeapRegionSize=8M -XX:G1ReservePercent=20 -XX:G1HeapWastePercent=5 -XX:G1MixedGCCountTarget=4 -XX:InitiatingHeapOccupancyPercent=15 -XX:G1MixedGCLiveThresholdPercent=90 -XX:G1RSetUpdatingPauseTimePercent=5 -XX:SurvivorRatio=32 -XX:+PerfDisableSharedMem -XX:MaxTenuringThreshold=1 -Dusing.aikars.flags=https://mcflags.emc.gs -Daikars.new.flags=true --add-modules=jdk.incubator.vector -jar server.jar --nogui
```

> Ajusta `-Xms` y `-Xmx` al mismo valor para evitar que la JVM redimensione el heap durante el juego.

### ZGC Para servidores de alto trafico

Generational ZGC esta disponible desde Java 21 y es ideal para servidores con 150+ jugadores o alta tasa de allocacion de memoria:

```bash
java -Xms10G -Xmx10G -XX:+UseZGC -XX:+ZGenerational -XX:+AlwaysPreTouch -XX:+DisableExplicitGC -XX:+UnlockExperimentalVMOptions --add-modules=jdk.incubator.vector -jar server.jar --nogui
```

> No mezcles flags de G1GC con ZGC. Son colectores completamente diferentes. ZGC tiene pausas de sub-milisegundo pero mayor overhead de CPU. Usa `/spark gcmonitor` para medir antes de cambiar.

### Comparativa G1GC vs ZGC 2026

| Metrica | G1GC | ZGC Generacional |
|---------|------|-----------------|
| Pausas GC | 50-200ms | menos de 1ms |
| Throughput | Alto | Moderado |
| Uso CPU | Moderado | Mayor |
| RAM minima | 4GB | 8GB+ |
| Cuando usarlo | Mayoria de servidores | 150+ jugadores, alto lag por GC |

### Generador de Flags Automatico

Usa [flags.sh](https://flags.sh) para generar las flags correctas segun tu configuracion de RAM y tipo de servidor.

### Flag Bonus SIMD para Pufferfish

```bash
--add-modules=jdk.incubator.vector
```

Agrega esta flag **antes** de `-jar`. Permite a Pufferfish usar instrucciones SIMD en tu CPU, haciendo el renderizado de mapas en plugins hasta 8x mas rapido.

---

## Plugins Mitos y Realidades

### Plugins que Evitar

#### Plugins de limpieza de items del suelo

Completamente innecesarios. Reemplazalos con:

- [`merge-radius`](#merge-radius) en spigot.yml
- [`alt-item-despawn-rate`](#alt-item-despawn-rate) en paper-world.yml

Los plugins de limpieza escanean constantemente el mundo, consumiendo mas recursos que simplemente no generar items en exceso.

#### Mob Stacker Plugins

Apilar entidades con spawn natural causa mas lag de lo que previene. El servidor intenta constantemente generar mas mobs para reemplazar los stacks. El unico caso aceptable es en spawners con cantidades masivas en servidores especializados.

#### Plugins que activan o desactivan otros plugins

Extremadamente peligrosos. Cargar un plugin en runtime puede causar errores fatales en datos. Desactivar uno puede romper dependencias.

> Esto incluye el comando `/reload`. [Lee por que es una mala idea](https://madelinemiller.dev/blog/problem-with-reload/).

#### EssentialsX y similares todo-en-uno

EssentialsX carga docenas de caracteristicas que probablemente no usas, muchas deprecated. Busca plugins especializados para las funciones que realmente necesitas en [SpigotMC](https://www.spigotmc.org/resources/) o [Modrinth](https://modrinth.com/plugins).

#### Plugins Anti-Lag y Optimizadores Magicos

`LagRemover`, `UltimateLagFixer` y similares son, en el mejor caso, inutiles y en el peor, perjudiciales. Todos los ajustes que hacen pueden hacerse manualmente con mejor control.

### Plugins Recomendados

| Plugin | Proposito | Enlace |
|--------|-----------|--------|
| **Spark** | Profiling de CPU y memoria en tiempo real | [spark.lucko.me](https://spark.lucko.me) |
| **Chunky** | Pre-generacion de chunks eficiente | [modrinth.com/plugin/chunky](https://modrinth.com/plugin/chunky) |
| **Plan** | Estadisticas de rendimiento y jugadores | [planplugin.com](https://planplugin.com) |
| **FarmControl** | Limitar entidades en granjas | [modrinth.com](https://modrinth.com) |

---

## Midiendo el Rendimiento

### MSPT — La Metrica Basica

```
/mspt
```

El comando `/mspt` de Paper muestra cuanto tiempo tardo el servidor en calcular los ultimos ticks.

| Valor MSPT | Estado |
|-----------|--------|
| menos de 50ms (ambos primeros) | Servidor saludable (20 TPS) |
| 50-100ms | Bajo 20 TPS, investigar |
| mas de 100ms | Servidor con lag serio |

El **tercer valor** es el pico maximo — valores altos ocasionales son normales.

### Spark — El Profiler Esencial

[Spark](https://spark.lucko.me/) es la herramienta mas importante para diagnosticar lag en profundidad.

```bash
/spark profiler start
/spark profiler stop
/spark gcmonitor
/spark health
```

Spark muestra exactamente que codigo esta consumiendo mas CPU, permitiendote identificar el plugin o sistema responsable del lag.

> [Guia completa para encontrar lag spikes con Spark](https://spark.lucko.me/docs/guides/Finding-lag-spikes)

### Timings (Legacy)

```bash
/timings paste
```

La herramienta basica de diagnostico integrada. Tiene un impacto serio en el rendimiento del servidor durante la medicion. Usa Spark siempre que sea posible.

Si usas Purpur o Pufferfish, puedes desactivar Timings completamente:

```yaml
# paper-global.yml
timings:
  enabled: false
```

### Interpretando los Resultados

```
TPS = Ticks Per Second (maximo 20)
MSPT = Milliseconds Per Tick (objetivo: menos de 50ms)

TPS = 20 cuando 20 x MSPT <= 1000ms
```

Si tu MSPT promedio esta cerca de 50ms, el servidor esta al limite. Identifica el culpable:

1. **Muchos mobs** — Reducir spawn-limits, activar DAB
2. **Chunks** — Reducir view y simulation distance
3. **Plugins** — Usar Spark para identificar el culpable
4. **Redstone** — Activar ALTERNATE_CURRENT
5. **Hopper lag** — Aumentar hopper-transfer y hopper-check

---

## Exploits y Como Solucionarlos

Los exploits pueden causar picos de lag devastadores o incluso crashear el servidor. Revisa la guia completa:

**[Minecraft Exploits and How to Fix Them](https://github.com/zMarkitos/minecraft-exploits-and-how-to-fix-them)**

### Los mas comunes en 2026

| Exploit | Causa | Solucion |
|---------|-------|----------|
| **TNT Dupers** | Miles de entidades TNT | `block-explosion-drop-decay: true` |
| **Carpet Bombing** | Colocar y romper bloques rapido | Rate limiting de bloques |
| **0-Tick Farms** | Bugs en actualizaciones de bloques | Paper los corrige automaticamente |
| **AFK Fish Farms** | Lag por cantidad de peces | Limitar despawn de flechas y peces |
| **End Crystal Lag** | Cristales del End en bucle | Limitar `entity-per-chunk-save-limit` |
| **Treasure Map Lag** | Busqueda en chunks no generados | `treasure-maps.enabled: false` |
| **Nether Roof Exploit** | Jugadores sobre el techo | `nether-ceiling-void-damage-height: 127` |

---

## FAQ

<details>
<summary><b>Cuanta RAM necesita mi servidor?</b></summary>

| Jugadores | RAM Recomendada |
|-----------|----------------|
| 1-10 | 4-6 GB |
| 10-30 | 6-8 GB |
| 30-60 | 8-12 GB |
| 60+ | 12-16 GB y considera Folia |

No asignes mas de 12-16GB sin medir primero. G1GC escala bien hasta aproximadamente 12GB.

</details>

<details>
<summary><b>Cual es la diferencia entre TPS y MSPT?</b></summary>

- **TPS (Ticks Per Second):** Cuantos ticks completa el servidor por segundo. Maximo 20. Bajo 20 hay lag perceptible.
- **MSPT (Milliseconds Per Tick):** Cuanto tiempo tarda cada tick. Maximo deseable: 50ms. Si un tick tarda mas de 50ms, el siguiente se retrasa, bajando el TPS.

Son inversamente relacionados: MSPT alto produce TPS bajo.

</details>

<details>
<summary><b>Por que no hay una guia para servidores Modded?</b></summary>

Los servidores modded (Forge, Fabric con mods) tienen su propia ecologia de optimizacion con mods como Starlight, Lithium, FerriteCore, etc. Esta guia esta especializada en el ecosistema de plugins (Paper/Bukkit). Para modded, busca guias especificas de [Fabric](https://fabricmc.net) o [NeoForge](https://neoforged.net).

</details>

<details>
<summary><b>Puedo usar estas configuraciones en 1.19.x?</b></summary>

La mayoria de configuraciones aplican a 1.19 - 1.21.x, aunque algunos valores especificos pueden variar. Siempre verifica la documentacion oficial del software que uses.

</details>

---

## Contribuir

Encontraste informacion incorrecta o quieres agregar algo?

1. Haz un **fork** del repositorio
2. Crea una rama: `git checkout -b mejora/mi-aporte`
3. Realiza tus cambios y haz commit: `git commit -m 'Agrega: descripcion del cambio'`
4. Push: `git push origin mejora/mi-aporte`
5. Abre un **Pull Request**

Tambien puedes abrir un **Issue** si encuentras errores o tienes sugerencias.

---

## Fuentes

| Fuente | Descripcion |
|--------|-------------|
| [zMarkitos_/zMOptimization_Minecraft_2026](https://github.com/zMarkitos_/zMOptimization_Minecraft_2026) | Guia original base |
| [PaperMC Documentation](https://docs.papermc.io) | Documentacion oficial de Paper |
| [Pufferfish Optimization Guide](https://docs.pufferfish.host/optimization/) | Guia oficial de Pufferfish |
| [Purpur Documentation](https://purpurmc.org/docs/) | Documentacion oficial de Purpur |
| [Aikar's JVM Flags](https://docs.papermc.io/paper/aikars-flags) | Flags de JVM estandar de la industria |
| [flags.sh](https://flags.sh) | Generador de startup flags |
| [Spark](https://spark.lucko.me/docs/) | Documentacion del profiler |
| [Obydux/Minecraft-startup-flags](https://github.com/Obydux/Minecraft-startup-flags) | Benchmarks modernos de JVM flags |

---

<div align="center">

Hecho para la comunidad de Minecraft en espanol

[![GitHub](https://img.shields.io/badge/GitHub-zMarkitos-black?style=for-the-badge&logo=github)](https://github.com/zMarkitos)

<br>

</div>
