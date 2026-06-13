<div align="center">

<img src="https://raw.githubusercontent.com/spectrasonic117/minecraft-optimization/main/assets/banner.png" alt="Minecraft Optimization Guide" width="100%">

# 🚀 Guía de Optimización de Servidores Minecraft
### La guía más completa y actualizada para 2026

[![Versión](https://img.shields.io/badge/Versión-2026.1-brightgreen?style=for-the-badge&logo=minecraft)](https://github.com)
[![Minecraft](https://img.shields.io/badge/Minecraft-1.20.x--1.21.x-blue?style=for-the-badge&logo=minecraft)](https://minecraft.net)
[![Paper](https://img.shields.io/badge/Paper-Compatible-orange?style=for-the-badge)](https://papermc.io)
[![Java](https://img.shields.io/badge/Java-21+-red?style=for-the-badge&logo=openjdk)](https://adoptium.net)
[![Licencia](https://img.shields.io/badge/Licencia-MIT-purple?style=for-the-badge)](LICENSE)

<br>

> **⚡ El rendimiento de tu servidor no depende solo del hardware.**  
> La configuración correcta puede ser la diferencia entre 20 TPS estables y un servidor que colapsa con 10 jugadores.

<br>

[📖 Introducción](#-introducción) •
[🛠️ Preparativos](#%EF%B8%8F-preparativos) •
[⚙️ Configuración](#%EF%B8%8F-configuraciones) •
[☕ Java Flags](#-java-startup-flags) •
[🔌 Plugins](#-plugins-mitos-y-realidades) •
[📊 Diagnóstico](#-midiendo-el-rendimiento) •
[🔒 Exploits](#-exploits-y-cómo-solucionarlos)

</div>

---

## 📋 Tabla de Contenidos

- [📖 Introducción](#-introducción)
- [🛠️ Preparativos](#%EF%B8%8F-preparativos)
  - [Elección del JAR de servidor](#-elección-del-jar-de-servidor)
  - [Pre-generación del mapa](#-pre-generación-del-mapa)
  - [Redes y Proxies](#-redes-y-proxies)
- [⚙️ Configuraciones](#%EF%B8%8F-configuraciones)
  - [Networking](#-networking)
  - [Chunks](#-chunks)
  - [Mobs](#-mobs)
  - [Misceláneos](#-misceláneos)
  - [Helpers & Seguridad](#-helpers--seguridad)
- [☕ Java Startup Flags](#-java-startup-flags)
  - [G1GC (Recomendado)](#g1gc-recomendado-para-la-mayoría)
  - [ZGC (Avanzado)](#zgc-para-servidores-de-alto-tráfico)
- [🔌 Plugins: Mitos y Realidades](#-plugins-mitos-y-realidades)
- [📊 Midiendo el Rendimiento](#-midiendo-el-rendimiento)
- [🔒 Exploits y Cómo Solucionarlos](#-exploits-y-cómo-solucionarlos)
- [❓ FAQ](#-faq)
- [🤝 Contribuir](#-contribuir)
- [📚 Fuentes](#-fuentes)

---

## 📖 Introducción

> **⚠️ Nota importante para usuarios de Vanilla, Fabric o Spigot:**  
> Ve a `server.properties` y cambia `sync-chunk-writes` a `false`. En Paper y sus forks esto está forzado a `false`, pero en otras implementaciones debes cambiarlo manualmente. Esto permite guardar chunks fuera del hilo principal, reduciendo drásticamente la carga del bucle principal.

**Nunca existirá una configuración perfecta universal.** Cada servidor tiene sus propias necesidades y límites. Esta guía pretende ayudarte a entender qué opciones tienen impacto real en el rendimiento y qué cambian exactamente, para que puedas tomar decisiones informadas.

### 🎯 ¿Qué aprenderás?

| Categoría | Beneficio Esperado |
|-----------|-------------------|
| 🖥️ Software correcto | Hasta **3x más rendimiento** vs Vanilla/Spigot |
| ⚙️ Configuración de chunks | Reducción del **40-60%** en uso de CPU |
| 🧟 Optimización de mobs | Mejora de **2-5 TPS** en servidores cargados |
| ☕ Java Flags | Eliminación del **80-90%** de lag spikes por GC |
| 🔌 Plugins correctos | Evitar pérdidas de hasta **5 TPS** por plugins malos |

---

## 🛠️ Preparativos

### 📦 Elección del JAR de Servidor

La elección del software es **la decisión más importante**. Aquí el panorama completo en 2026:

#### ✅ Software Recomendado

```
Vanilla → Spigot → Paper → Pufferfish → Purpur
         (cada fork hereda las optimizaciones del anterior)
```

| Software | Descripción | ¿Para quién? | Enlace |
|----------|-------------|--------------|--------|
| 🥇 **Paper** | El fork más popular. Corrige inconsistencias de gameplay y mejora el rendimiento significativamente. Base de toda la jerarquía. | La mayoría de servidores | [papermc.io](https://papermc.io) |
| 🥈 **Pufferfish** | Fork de Paper enfocado en rendimiento puro. Incluye DAB (Dynamic Activation Brain), spawn asíncrono de mobs, SIMD para mapas y más. | Servidores con 50+ jugadores o granjas pesadas | [pufferfish.host](https://pufferfish.host) |
| 🥉 **Purpur** | Fork de Pufferfish. Incluye todas las optimizaciones + cientos de opciones de personalización del gameplay. Las funciones extra están **desactivadas por defecto**. | Servidores que necesitan personalización extrema | [purpurmc.org](https://purpurmc.org) |
| ⚡ **Folia** | Fork experimental de Paper con multithreading por regiones. Rompe muchos plugins pero puede manejar cargas masivas. | Servidores con 150+ jugadores, uso avanzado | [papermc.io/folia](https://papermc.io/software/folia) |

> **💡 Recomendación 2026:** Para un SMP público o servidor de supervivencia con plugins, usa **Purpur** o **Pufferfish**. Para una red grande, considera **Folia** solo si tus plugins lo soportan.

#### ❌ Software a Evitar

> **🚫 NUNCA uses estos:**

- **Bukkit / CraftBukkit / Spigot** — Extremadamente anticuados en rendimiento. No hay razón para usarlos en 2026.
- **JARs de pago que prometen "async todo"** — 99.99% son estafas o implementaciones rotas.
- **Forks exóticos downstream de Purpur** — Casi siempre tienen problemas de estabilidad sin beneficio real de rendimiento.
- **Versiones inestables/snapshot** — Si quieres rendimiento, usa releases estables.

---

### 🗺️ Pre-generación del Mapa

La pre-generación en 2026 ya **no es estrictamente necesaria** en CPUs modernas, pero sigue siendo útil si:

- Usas plugins de mapas del mundo como [Pl3xMap](https://github.com/jpenilla/Pl3xMap) o [Dynmap](https://github.com/webbukkit/dynmap)
- Tienes una CPU muy limitada de un solo hilo
- Quieres eliminar completamente el lag de generación de chunks nuevos

**Plugin recomendado:** [Chunky](https://modrinth.com/plugin/chunky)

```
⚠️ IMPORTANTE: Configura siempre un borde de mundo ANTES de pregenerar
/worldborder set [diametro]
```

> **El Overworld, Nether y End tienen bordes de mundo separados.** El Nether es 1/8 del tamaño del Overworld. Configura cada dimensión correctamente o los jugadores pueden terminar fuera del borde.

> **El borde de mundo vanilla también limita el rango de búsqueda de mapas del tesoro**, que puede causar picos de lag masivos si no está configurado.

---

### 🌐 Redes y Proxies

Si administras una **red de servidores**, la elección del proxy es crucial:

| Proxy | Estado en 2026 | Recomendación |
|-------|---------------|---------------|
| **BungeeCord** | Funcional pero legacy | Solo si necesitas plugins específicos sin alternativa |
| **Waterfall** | Fork de BungeeCord, más estable | Alternativa si migras desde BungeeCord |
| **Velocity** ⭐ | Moderno, alto rendimiento, activamente desarrollado | **RECOMENDADO para cualquier red nueva** |

**Velocity**, desarrollado por el equipo de PaperMC, es la única opción real para redes nuevas en 2026. Soporta 1000+ jugadores con degradación mínima de rendimiento, tiene seguridad mejorada y una API moderna.

---

## ⚙️ Configuraciones

> **💡 Consejo:** Realiza cambios de a uno y mide el impacto con [Spark](#-spark) antes de continuar.

---

### 🌍 Networking

#### `server.properties`

<details>
<summary><b>network-compression-threshold</b> — <code>Valor recomendado: 256</code></summary>

```properties
network-compression-threshold=256
```

Define el tamaño mínimo de un paquete antes de que el servidor lo comprima.

- **Más alto** → Menos CPU, más ancho de banda
- **`-1`** → Desactiva la compresión (ideal si el proxy está en la misma máquina con <2ms de ping)
- **Más bajo** → Más CPU, menos ancho de banda (perjudica conexiones lentas)

</details>

#### `purpur.yml`

<details>
<summary><b>use-alternate-keepalive</b> — <code>Valor recomendado: true</code></summary>

```yaml
settings:
  use-alternate-keepalive: true
```

Envía un paquete keepalive por segundo al jugador. Solo expulsa al jugador si **ninguno** fue respondido en 30 segundos. Reduce desconexiones en jugadores con conexiones inestables.

> ⚠️ Incompatible con **TCPShield**.

</details>

---

### 📦 Chunks

#### `server.properties`

<details>
<summary><b>simulation-distance</b> — <code>Valor recomendado: 4</code></summary>

```properties
simulation-distance=4
```

La distancia (en chunks) alrededor del jugador donde las cosas **realmente suceden**: hornos, cultivos, árboles, mobs, etc.

Mantenerlo bajo (3-4) es intencional. Se complementa con `view-distance` para que los jugadores vean más lejos sin el mismo impacto en rendimiento.

</details>

<details>
<summary><b>view-distance</b> — <code>Valor recomendado: 7</code></summary>

```properties
view-distance=7
```

Chunks enviados visualmente al cliente. La distancia total será el mayor valor entre `simulation-distance` y `view-distance`.

> Ejemplo: simulation=4, view=7 → el cliente ve 7 chunks, pero solo 4 están simulados activamente.

</details>

#### `paper-world.yml`

<details>
<summary><b>delay-chunk-unloads-by</b> — <code>Valor recomendado: 10s</code></summary>

```yaml
chunks:
  delay-chunk-unloads-by: 10s
```

Mantiene los chunks cargados 10 segundos después de que un jugador se aleje. Evita cargar/descargar constantemente los mismos chunks cuando un jugador va y viene.

</details>

<details>
<summary><b>max-auto-save-chunks-per-tick</b> — <code>Valor recomendado: 8</code></summary>

```yaml
chunks:
  max-auto-save-chunks-per-tick: 8
```

Distribuye el guardado de mundo en el tiempo para mejor rendimiento promedio. Con 20-30+ jugadores, considera aumentarlo a `12` o `16`.

</details>

<details>
<summary><b>prevent-moving-into-unloaded-chunks</b> — <code>Valor recomendado: true</code></summary>

```yaml
chunks:
  prevent-moving-into-unloaded-chunks: true
```

Evita que los jugadores entren en chunks no cargados, previniendo cargas de sincronización que atascan el hilo principal.

</details>

<details>
<summary><b>entity-per-chunk-save-limit</b> — <code>Ver valores recomendados</code></summary>

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

Limita cuántas entidades de cada tipo pueden guardarse por chunk. Previene crashes al cargar chunks con cantidades absurdas de proyectiles guardados.

</details>

#### `pufferfish.yml`

<details>
<summary><b>max-loads-per-projectile</b> — <code>Valor recomendado: 8</code></summary>

```yaml
max-loads-per-projectile: 8
```

Limita cuántos chunks puede cargar un proyectil durante su trayectoria. Reduce el lag de enderpearls y tridentes.

</details>

---

### 🧟 Mobs

#### `bukkit.yml`

<details>
<summary><b>spawn-limits</b> — <code>Ver valores recomendados</code></summary>

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

La fórmula es: `[jugadores activos] × [límite]`. A menor número, menos trabajo para el servidor. Combinar con `per-player-mob-spawns: true` en Paper para distribución equitativa.

</details>

<details>
<summary><b>ticks-per</b> — <code>Ver valores recomendados</code></summary>

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

Frecuencia (en ticks) con la que el servidor intenta generar cada tipo de entidad. Los animales/agua no necesitan aparecer cada tick, ya que no mueren tan rápido.

</details>

#### `spigot.yml`

<details>
<summary><b>mob-spawn-range</b> — <code>Valor recomendado: 3</code></summary>

```yaml
mob-spawn-range: 3
```

Rango en chunks donde pueden aparecer mobs alrededor del jugador. Debe ser ≤ simulation-distance y nunca mayor que el rango de despawn / 16.

</details>

<details>
<summary><b>entity-activation-range</b> — <code>Ver valores recomendados</code></summary>

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
<summary><b>entity-tracking-range</b> — <code>Ver valores recomendados</code></summary>

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
<summary><b>nerf-spawner-mobs</b> — <code>Valor recomendado: true</code></summary>

```yaml
nerf-spawner-mobs: true
```

Los mobs generados por spawners no tendrán IA. Drásticamente mejora el rendimiento en servidores con muchos spawners. Los mobs pueden saltar en el agua si `spawner-nerfed-mobs-should-jump: true` en paper-world.

</details>

#### `paper-world.yml`

<details>
<summary><b>despawn-ranges</b> — <code>Ver valores recomendados</code></summary>

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
- **Hard range:** despawn instantáneo
- Fórmula recomendada: `(simulation-distance × 16) + 8`

</details>

<details>
<summary><b>per-player-mob-spawns</b> — <code>Valor recomendado: true</code></summary>

```yaml
entities:
  spawning:
    per-player-mob-spawns: true
```

Distribuye el spawn de mobs equitativamente entre jugadores. Evita que un jugador con una granja enorme acapare toda la mob cap.

</details>

<details>
<summary><b>max-entity-collisions</b> — <code>Valor recomendado: 2</code></summary>

```yaml
entities:
  max-entity-collisions: 2
```

Limita cuántas colisiones puede procesar una entidad por tick. `0` = imposible empujar entidades. `2` es suficiente para la mayoría de casos.

</details>

<details>
<summary><b>update-pathfinding-on-block-update</b> — <code>Valor recomendado: false</code></summary>

```yaml
entities:
  update-pathfinding-on-block-update: false
```

Reduce el pathfinding innecesario. Los mobs actualizarán su ruta pasivamente cada 5 ticks en lugar de en cada actualización de bloque.

</details>

#### `pufferfish.yml` — DAB (Dynamic Activation Brain)

> **🧠 Una de las optimizaciones más importantes de Pufferfish.**

<details>
<summary><b>dab.enabled</b> — <code>Valor recomendado: true</code></summary>

```yaml
dab:
  enabled: true
  max-tick-freq: 20
  activation-dist-mod: 7
```

DAB reduce cuánto "piensa" una entidad basándose en qué tan lejos está de los jugadores. A diferencia de EAR (que hace un corte brusco), DAB trabaja en gradiente:

- **Entidades cercanas** → tick completo
- **Entidades lejanas** → tick reducido progresivamente

| Opción | Descripción | Recomendado |
|--------|-------------|-------------|
| `max-tick-freq` | Velocidad más lenta para entidades lejanas | `20` |
| `activation-dist-mod` | Qué tan cerca del jugador activa DAB | `7` |

> ⚠️ Si DAB rompe tus granjas, prueba aumentando `activation-dist-mod` o reduciendo `max-tick-freq`.

</details>

<details>
<summary><b>enable-async-mob-spawning</b> — <code>Valor recomendado: true</code></summary>

```yaml
enable-async-mob-spawning: true
```

Mueve el cálculo computacional del spawn de mobs a un hilo separado. Requiere `per-player-mob-spawns: true` en Paper. **No genera mobs de forma asíncrona**, solo optimiza el cálculo previo.

</details>

#### `purpur.yml`

<details>
<summary><b>villager.lobotomize.enabled</b> — <code>Valor recomendado: true (si hay lag de aldeanos)</code></summary>

```yaml
mobs:
  villager:
    lobotomize:
      enabled: true
      check-interval: 100
```

Los aldeanos que no pueden seguir una ruta pierden su IA y solo reabastecen ofertas periódicamente. **Activa solo si los aldeanos causan lag.**

</details>

<details>
<summary><b>entities-can-use-portals</b> — <code>Valor recomendado: false</code></summary>

```yaml
settings:
  entities-can-use-portals: false
```

Previene que entidades (no jugadores) usen portales y carguen chunks al cambiar dimensiones.

</details>

---

### 🔧 Misceláneos

#### `spigot.yml`

<details>
<summary><b>merge-radius</b> — <code>Ver valores recomendados</code></summary>

```yaml
merge-radius:
  item: 3.5
  exp: 4.0
```

Distancia a la que ítems y orbes de exp se fusionan entre sí, reduciendo entidades en el suelo. Muy alto puede hacer que ítems "teletransporten" a través de paredes.

</details>

<details>
<summary><b>hopper-transfer / hopper-check</b> — <code>Valor recomendado: 8</code></summary>

```yaml
hopper-transfer: 8
hopper-check: 8
```

Ticks de espera entre acciones de tolvas. Mejora significativamente el rendimiento con muchas tolvas. Valores muy altos pueden romper relojes de tolvas.

</details>

#### `paper-world.yml`

<details>
<summary><b>alt-item-despawn-rate</b> — <code>Ver valores recomendados</code></summary>

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

Ítems comunes y sin valor desaparecen más rápido, reduciendo entidades en el suelo. Alternativa eficiente a plugins de limpieza.

</details>

<details>
<summary><b>redstone-implementation</b> — <code>Valor recomendado: ALTERNATE_CURRENT</code></summary>

```yaml
redstone:
  implementation: ALTERNATE_CURRENT
```

Reemplaza el sistema de redstone vanilla con una implementación más rápida basada en el mod [Alternate Current](https://modrinth.com/mod/alternate-current). Reduce actualizaciones redundantes de bloques.

> ⚠️ Puede introducir inconsistencias menores con redstone muy técnica, pero la ganancia de rendimiento supera ampliamente los posibles problemas.

</details>

<details>
<summary><b>optimize-explosions</b> — <code>Valor recomendado: true</code></summary>

```yaml
misc:
  optimize-explosions: true
```

Reemplaza el algoritmo de explosión vanilla por uno más rápido con mínima diferencia de cálculo de daño.

</details>

<details>
<summary><b>treasure-maps.enabled</b> — <code>Valor recomendado: false</code></summary>

```yaml
feature-seeds:
  # ...
environment:
  treasure-maps:
    enabled: false
    find-already-discovered:
      loot-tables: true
      villager-trade: true
```

Generar mapas del tesoro es **extremadamente caro** y puede colgar el servidor si la estructura está en chunks no generados. Solo activa si has pregenerado el mundo completamente.

</details>

<details>
<summary><b>non-player-arrow-despawn-rate / creative-arrow-despawn-rate</b> — <code>Valor recomendado: 20</code></summary>

```yaml
entities:
  spawning:
    non-player-arrow-despawn-rate: 20
    creative-arrow-despawn-rate: 20
```

Las flechas de mobs y modo creativo desaparecen en 1 segundo (20 ticks) tras impactar. Los jugadores no pueden recogerlas de todas formas.

</details>

---

### 🛡️ Helpers & Seguridad

#### `paper-world.yml`

<details>
<summary><b>anti-xray.enabled</b> — <code>Valor recomendado: true</code></summary>

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

Mucho más eficiente que cualquier plugin anti-xray. El impacto en rendimiento es mínimo.

- **Mode 1:** Oculta ores específicos
- **Mode 2 (Recomendado):** Reemplaza bloques con falsos ores → más seguro pero más uso de CPU

</details>

<details>
<summary><b>nether-ceiling-void-damage-height</b> — <code>Valor recomendado: 127</code></summary>

```yaml
environment:
  nether-ceiling-void-damage-height: 127
```

Daño de vacío sobre el techo del Nether (altura 128). Evita que los jugadores exploten el techo del Nether. Si modificas la altura del Nether: `[tu_altura_nether] - 1`.

</details>

---

## ☕ Java Startup Flags

> **⚠️ Minecraft 1.20.4+ requiere Java 21.**  
> Proveedores recomendados: [Adoptium](https://adoptium.net/) y [Amazon Corretto](https://aws.amazon.com/corretto/)  
> **NO uses Oracle JDK** — cambios de licencia en 2023 lo hacen inadecuado para producción.

### G1GC (Recomendado para la mayoría)

**G1GC** sigue siendo la mejor opción para la mayoría de servidores en 2026. Las flags de Aikar son el estándar de la industria:

```bash
java -Xms8G -Xmx8G \
  -XX:+UseG1GC \
  -XX:+ParallelRefProcEnabled \
  -XX:MaxGCPauseMillis=200 \
  -XX:+UnlockExperimentalVMOptions \
  -XX:+DisableExplicitGC \
  -XX:+AlwaysPreTouch \
  -XX:G1NewSizePercent=30 \
  -XX:G1MaxNewSizePercent=40 \
  -XX:G1HeapRegionSize=8M \
  -XX:G1ReservePercent=20 \
  -XX:G1HeapWastePercent=5 \
  -XX:G1MixedGCCountTarget=4 \
  -XX:InitiatingHeapOccupancyPercent=15 \
  -XX:G1MixedGCLiveThresholdPercent=90 \
  -XX:G1RSetUpdatingPauseTimePercent=5 \
  -XX:SurvivorRatio=32 \
  -XX:+PerfDisableSharedMem \
  -XX:MaxTenuringThreshold=1 \
  -Dusing.aikars.flags=https://mcflags.emc.gs \
  -Daikars.new.flags=true \
  --add-modules=jdk.incubator.vector \
  -jar server.jar --nogui
```

> **💡 Ajusta `-Xms` y `-Xmx` al mismo valor** para evitar que la JVM redimensione el heap durante el juego.

### ZGC (Para servidores de alto tráfico)

**Generational ZGC** está disponible desde Java 21 y es ideal para servidores con 150+ jugadores o alta tasa de allocación de memoria:

```bash
java -Xms10G -Xmx10G \
  -XX:+UseZGC \
  -XX:+ZGenerational \
  -XX:+AlwaysPreTouch \
  -XX:+DisableExplicitGC \
  -XX:+UnlockExperimentalVMOptions \
  --add-modules=jdk.incubator.vector \
  -jar server.jar --nogui
```

> ⚠️ **NO mezcles flags de G1GC con ZGC.** Son colectores completamente diferentes. ZGC tiene pausas de sub-milisegundo pero mayor overhead de CPU. Usa `/spark gcmonitor` para medir antes de cambiar.

### Comparativa G1GC vs ZGC (2026)

| Métrica | G1GC | ZGC Generacional |
|---------|------|-----------------|
| Pausas GC | 50-200ms | <1ms |
| Throughput | Alto | Moderado |
| Uso CPU | Moderado | Mayor |
| RAM mínima | 4GB | 8GB+ |
| ¿Cuándo usarlo? | Mayoría de servidores | 150+ jugadores, alto lag por GC |

### 🛠️ Generador de Flags Automático

Usa [flags.sh](https://flags.sh) para generar las flags correctas según tu configuración de RAM y tipo de servidor.

### ⚡ Flag Bonus: SIMD para Pufferfish

```bash
--add-modules=jdk.incubator.vector
```

Agrega esta flag **antes** de `-jar`. Permite a Pufferfish usar instrucciones SIMD en tu CPU, haciendo el renderizado de mapas en plugins hasta **8x más rápido**.

---

## 🔌 Plugins: Mitos y Realidades

### ❌ Plugins que EVITAR

#### 🗑️ Plugins de limpieza de ítems del suelo
**Completamente innecesarios.** Reemplázalos con:
- [`merge-radius`](#merge-radius) en spigot.yml
- [`alt-item-despawn-rate`](#alt-item-despawn-rate) en paper-world.yml

Los plugins de limpieza escanean constantemente el mundo, consumiendo más recursos que simplemente no generar ítems excess.

#### 📚 Mob Stacker Plugins
Apilar entidades con spawn natural **causa más lag** de lo que previene. El servidor intenta constantemente generar más mobs para reemplazar los "stacks". El único caso aceptable es en spawners con cantidades masivas en servidores especializados.

#### 🔄 Plugins que activan/desactivan otros plugins
**Extremadamente peligrosos.** Cargar un plugin en runtime puede causar errores fatales en datos. Desactivar uno puede romper dependencias.

> Esto incluye el comando `/reload`. [Lee por qué es una mala idea](https://madelinemiller.dev/blog/problem-with-reload/).

#### 🧩 EssentialsX (y similares todo-en-uno)
EssentialsX carga docenas de características que probablemente no usas, muchas deprecated. En su lugar, busca plugins especializados para las funciones que realmente necesitas en [SpigotMC](https://www.spigotmc.org/resources/) o [Modrinth](https://modrinth.com/plugins).

#### 🚫 Plugins "Anti-Lag" y "Optimizadores Mágicos"
`LagRemover`, `UltimateLagFixer`, y similares son, en el mejor caso, inútiles y en el peor, perjudiciales. Todos los ajustes que hacen pueden hacerse manualmente con mejor control.

### ✅ Plugins RECOMENDADOS

| Plugin | Propósito | Enlace |
|--------|-----------|--------|
| 🔥 **Spark** | Profiling de CPU y memoria en tiempo real | [spark.lucko.me](https://spark.lucko.me) |
| 🗺️ **Chunky** | Pre-generación de chunks eficiente | [modrinth.com/plugin/chunky](https://modrinth.com/plugin/chunky) |
| 📊 **Plan** | Estadísticas de rendimiento y jugadores | [planplugin.com](https://planplugin.com) |
| 🌾 **FarmControl** | Limitar entidades en granjas | [modrinth.com](https://modrinth.com) |

---

## 📊 Midiendo el Rendimiento

### 🎯 MSPT — La Métrica Básica

```
/mspt
```

El comando `/mspt` de Paper muestra cuánto tiempo tardó el servidor en calcular los últimos ticks.

| Valor MSPT | Estado |
|-----------|--------|
| < 50ms (ambos primeros) | ✅ Servidor saludable (20 TPS) |
| 50-100ms | ⚠️ Bajo 20 TPS, investigar |
| > 100ms | ❌ Servidor con lag serio |

El **tercer valor** es el pico máximo — valores altos ocasionales son normales.

### 🔥 Spark — El Profiler Esencial

[Spark](https://spark.lucko.me/) es la herramienta más importante para diagnosticar lag en profundidad.

```bash
# Iniciar profiling
/spark profiler start

# Detener y ver resultados
/spark profiler stop

# Monitor de GC
/spark gcmonitor

# Health report rápido
/spark health
```

Spark muestra exactamente qué código está consumiendo más CPU, permitiéndote identificar el plugin o sistema responsable del lag.

> 📖 [Guía completa para encontrar lag spikes con Spark](https://spark.lucko.me/docs/guides/Finding-lag-spikes)

### ⏱️ Timings (Legacy)

```bash
/timings paste
```

La herramienta básica de diagnóstico integrada. **Impacto serio en el rendimiento del servidor durante la medición.** Usa Spark siempre que sea posible.

Si usas Purpur o Pufferfish, puedes desactivar Timings completamente para mejor rendimiento:

```yaml
# paper-global.yml
timings:
  enabled: false
```

### 📈 Interpretando los Resultados

```
TPS = Ticks Per Second (máximo 20)
MSPT = Milliseconds Per Tick (objetivo: <50ms)

TPS = 20 → 20 × MSPT ≤ 1000ms → MSPT ≤ 50ms
```

Si tu MSPT promedio está cerca de 50ms, el servidor está al límite. Identifica el culpable:

1. **Muchos mobs** → Reducir spawn-limits, activar DAB
2. **Chunks** → Reducir view/simulation distance
3. **Plugins** → Usar Spark para identificar el culpable
4. **Redstone** → Activar ALTERNATE_CURRENT
5. **Hopper lag** → Aumentar hopper-transfer y hopper-check

---

## 🔒 Exploits y Cómo Solucionarlos

Los exploits pueden causar picos de lag devastadores o incluso crashear el servidor. Revisa la guía completa de exploits y soluciones:

**[➡️ Minecraft Exploits & How to Fix Them](https://github.com/YouHaveTrouble/minecraft-exploits-and-how-to-fix-them)**

### Los más comunes en 2026:

| Exploit | Causa | Solución |
|---------|-------|----------|
| **TNT Dupers** | Miles de entidades TNT | `block-explosion-drop-decay: true` |
| **Carpet Bombing** | Colocar/romper bloques rápido | Rate limiting de bloques |
| **0-Tick Farms** | Bugs en actualizaciones de bloques | Paper los corrige automáticamente |
| **AFK Fish Farms** | Lag por cantidad de peces | Limitar despawn de flechas y peces |
| **End Crystal Lag** | Cristales del End en bucle | Limitar `entity-per-chunk-save-limit` |
| **Treasure Map Lag** | Búsqueda en chunks no generados | `treasure-maps.enabled: false` |
| **Nether Roof Exploit** | Jugadores sobre el techo | `nether-ceiling-void-damage-height: 127` |

---

## ❓ FAQ

<details>
<summary><b>¿Cuánta RAM necesita mi servidor?</b></summary>

| Jugadores | RAM Recomendada |
|-----------|----------------|
| 1-10 | 4-6 GB |
| 10-30 | 6-8 GB |
| 30-60 | 8-12 GB |
| 60+ | 12-16 GB + considera Folia |

> ⚠️ **No asignes más de 12-16GB sin medir primero.** "Más RAM = mejor" deja de ser verdad a cierto punto. G1GC escala bien hasta ~12GB.

</details>

<details>
<summary><b>¿Cuál es la diferencia entre TPS y MSPT?</b></summary>

- **TPS (Ticks Per Second):** Cuántos ticks completa el servidor por segundo. Máximo 20. Bajo 20 = lag perceptible.
- **MSPT (Milliseconds Per Tick):** Cuánto tiempo tarda cada tick. Máximo deseable: 50ms. Si un tick tarda más de 50ms, el siguiente se retrasa, bajando el TPS.

Son inversamente relacionados: MSPT alto → TPS bajo.

</details>

<details>
<summary><b>¿Por qué no hacer una guía para servidores Modded?</b></summary>

Los servidores modded (Forge, Fabric+mods) tienen su propia ecología de optimización con mods como Starlight, Lithium, FerriteCore, etc. Esta guía está especializada en el ecosistema de plugins (Paper/Bukkit). Para modded, busca guías específicas de [Fabric](https://fabricmc.net) o [NeoForge](https://neoforged.net).

</details>

<details>
<summary><b>¿Puedo usar estas configuraciones en 1.19.x?</b></summary>

La mayoría de configuraciones aplican a **1.19 - 1.21.x**, aunque algunos valores específicos pueden variar. Siempre verifica la documentación oficial del software que uses.

</details>

---

## 🤝 Contribuir

¿Encontraste información incorrecta o quieres agregar algo?

1. **Fork** este repositorio
2. Crea una rama: `git checkout -b mejora/mi-aporte`
3. Realiza tus cambios y **commitea**: `git commit -m 'Agrega: descripción del cambio'`
4. **Push**: `git push origin mejora/mi-aporte`
5. Abre un **Pull Request**

También puedes abrir un **Issue** si encuentras errores o tienes sugerencias.

---

## 📚 Fuentes

Esta guía está basada en investigación de múltiples fuentes confiables:

| Fuente | Descripción |
|--------|-------------|
| [YouHaveTrouble/minecraft-optimization](https://github.com/YouHaveTrouble/minecraft-optimization) | Guía original base |
| [PaperMC Documentation](https://docs.papermc.io) | Documentación oficial de Paper |
| [Pufferfish Optimization Guide](https://docs.pufferfish.host/optimization/) | Guía oficial de Pufferfish |
| [Purpur Documentation](https://purpurmc.org/docs/) | Documentación oficial de Purpur |
| [Aikar's JVM Flags](https://docs.papermc.io/paper/aikars-flags) | Flags de JVM estándar de la industria |
| [flags.sh](https://flags.sh) | Generador de startup flags |
| [Spark](https://spark.lucko.me/docs/) | Documentación del profiler |
| [Obydux/Minecraft-startup-flags](https://github.com/Obydux/Minecraft-startup-flags) | Benchmarks modernos de JVM flags |

---

<div align="center">

**Hecho con ❤️ para la comunidad de Minecraft en español**

[![GitHub](https://img.shields.io/badge/GitHub-spectrasonic117-black?style=for-the-badge&logo=github)](https://github.com/spectrasonic117)
[![Twitter/X](https://img.shields.io/badge/Twitter-@spectrasonic117-blue?style=for-the-badge&logo=x)](https://x.com/spectrasonic117)

<br>

> Basado en el proyecto original de [YouHaveTrouble](https://github.com/YouHaveTrouble/minecraft-optimization)  
> Traducción, actualización 2026 y mejoras por [Spectrasonic](https://x.com/spectrasonic117)

<br>

⭐ **Si esta guía te fue útil, dale una estrella al repositorio** ⭐

</div>
