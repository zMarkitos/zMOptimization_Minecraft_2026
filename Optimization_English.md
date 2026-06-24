<div align="center">

<img src="https://raw.githubusercontent.com/zMarkitos/zMOptimization_Minecraft_2026/refs/heads/main/assets/banner.svg" alt="Minecraft Optimization Guide" width="100%">

# Minecraft Server Optimization Guide
### The most comprehensive and up-to-date guide for 2026

[![Version](https://img.shields.io/badge/Version-2026.1-brightgreen?style=for-the-badge)](https://github.com)
[![Minecraft](https://img.shields.io/badge/Minecraft-1.20.x--1.21.x-blue?style=for-the-badge)](https://minecraft.net)
[![Paper](https://img.shields.io/badge/Paper-Compatible-orange?style=for-the-badge)](https://papermc.io)
[![Java](https://img.shields.io/badge/Java-21+-red?style=for-the-badge)](https://adoptium.net)
[![License](https://img.shields.io/badge/License-MIT-purple?style=for-the-badge)](LICENSE)
[![ES](https://img.shields.io/badge/Lang-English-blue?style=for-the-badge)](README.en.md)

<br>

> **Your server's performance doesn't depend solely on hardware.**
> The right configuration can be the difference between stable 20 TPS and a server that crashes with 10 players.

<br>

[Introduction](#introduction) •
[Preparations](#preparations) •
[Configurations](#configurations) •
[Java Flags](#java-startup-flags) •
[Plugins](#plugins-myths-and-realities) •
[Performance Monitoring](#measuring-performance) •
[Exploits](#exploits-and-how-to-fix-them)

</div>

---

## Table of Contents

- [Introduction](#introduction)
- [Preparations](#preparations)
  - [Choosing the Server JAR](#choosing-the-server-jar)
  - [World Pre-generation](#world-pre-generation)
  - [Networks and Proxies](#networks-and-proxies)
- [Configurations](#configurations)
  - [Networking](#networking)
  - [Chunks](#chunks)
  - [Mobs](#mobs)
  - [Miscellaneous](#miscellaneous)
  - [Helpers and Security](#helpers-and-security)
- [Java Startup Flags](#java-startup-flags)
  - [Recommended G1GC](#g1gc-recommended-for-most)
  - [Advanced ZGC](#zgc-for-high-traffic-servers)
- [Plugins Myths and Realities](#plugins-myths-and-realities)
- [Measuring Performance](#measuring-performance)
- [Exploits and How to Fix Them](#exploits-and-how-to-fix-them)
- [FAQ](#faq)
- [Contributing](#contributing)
- [Sources](#sources)

---

## Introduction

> **Important note for Vanilla, Fabric, or Spigot users:**
> Go to `server.properties` and change `sync-chunk-writes` to `false`. In Paper and its forks, this is forced to `false`, but in other implementations, you must change it manually. This allows chunks to be saved outside the main thread, drastically reducing the load on the main loop.

**There will never be a perfect universal configuration.** Every server has its own needs and limits. This guide aims to help you understand which options have a real impact on performance and what exactly they change, so you can make informed decisions.

### What You Will Learn

| Category | Expected Benefit |
|-----------|-------------------|
| Correct Software | Up to 3x better performance vs Vanilla/Spigot |
| Chunk Configuration | 40-60% reduction in CPU usage |
| Mob Optimization | 2-5 TPS improvement on loaded servers |
| Java Flags | Elimination of 80-90% of lag spikes due to GC |
| Correct Plugins | Avoid losing up to 5 TPS from bad plugins |

---

## Preparations

### Choosing the Server JAR

Choosing the right software is **the most important decision**. Here is the complete landscape for 2026:

#### Recommended Software

```
Vanilla > Spigot > Paper > Pufferfish > Purpur
         (each fork inherits optimizations from the previous)
```

| Software | Description | Best For | Link |
|----------|-------------|------------|--------|
| **Paper** | The most popular fork. Fixes gameplay inconsistencies and significantly improves performance. The foundation for the entire hierarchy. | Most servers | [papermc.io](https://papermc.io) |
| **Pufferfish** | A Paper fork focused purely on performance. Includes DAB (Dynamic Activation Brain), asynchronous mob spawning, SIMD for maps, and more. | Servers with 50+ players or heavy farms | [pufferfish.host](https://pufferfish.host) |
| **Purpur** | A Pufferfish fork. Includes all optimizations plus hundreds of gameplay customization options. Extra features are disabled by default. | Servers needing extreme customization | [purpurmc.org](https://purpurmc.org) |
| **Folia** | An experimental Paper fork with region-based multithreading. Breaks many plugins but can handle massive loads. | Servers with 150+ players, advanced use | [papermc.io/folia](https://papermc.io/software/folia) |

> **2026 Recommendation:** For a public SMP or survival server with plugins, use **Purpur** or **Pufferfish**. For a large network, consider **Folia** only if your plugins support it.

#### Software to Avoid

- **Bukkit / CraftBukkit / Spigot** — Extremely outdated in terms of performance. No reason to use them in 2026.
- **Paid JARs that promise async everything** — 99.99% are scams or broken implementations.
- **Exotic forks downstream of Purpur** — Almost always have stability issues without any real performance benefit.
- **Unstable/snapshot versions** — If you want performance, use stable releases.

---

### World Pre-generation

World pre-generation is no longer strictly necessary on modern CPUs in 2026, but it remains useful if:

- You use world map plugins like [Pl3xMap](https://github.com/jpenilla/Pl3xMap) or [Dynmap](https://github.com/webbukkit/dynmap)
- You have a very limited single-thread CPU
- You want to completely eliminate lag from generating new chunks

**Recommended plugin:** [Chunky](https://modrinth.com/plugin/chunky)

```
IMPORTANT: Always set a world border BEFORE pre-generating
/worldborder set [diameter]
```

> **The Overworld, Nether, and End have separate world borders.** The Nether is 1/8 the size of the Overworld. Configure each dimension correctly, or players may end up outside the border.

> **The vanilla world border also limits treasure map search range**, which can cause massive lag spikes if not configured.

---

### Networks and Proxies

If you manage a **server network**, choosing the proxy is crucial:

| Proxy | Status in 2026 | Recommendation |
|-------|---------------|----------------|
| **BungeeCord** | Functional but legacy | Only if you need specific plugins without alternatives |
| **Waterfall** | BungeeCord fork, more stable | Alternative if migrating from BungeeCord |
| **Velocity** | Modern, high-performance, actively developed | RECOMMENDED for any new network |

**Velocity**, developed by the PaperMC team, is the only real choice for new networks in 2026. It supports 1000+ players with minimal performance degradation, has improved security, and a modern API.

---

## Configurations

> **Tip:** Make changes one at a time and measure the impact with [Spark](#spark) before continuing.

---

### Networking

#### `server.properties`

<details>
<summary><b>network-compression-threshold</b> — Recommended value: 256</summary>

```properties
network-compression-threshold=256
```

Defines the minimum packet size (in bytes) required for the server to compress a network packet before sending it.

Compression reduces bandwidth usage but slightly increases CPU usage. Small packets are not compressed to avoid unnecessary overhead.

* **Higher value** — Less CPU usage, more uncompressed network traffic
* **Lower value** — More compression, lower bandwidth usage, but higher CPU load
* **`-1`** — Disables compression entirely (all packets are sent uncompressed; this can significantly increase bandwidth usage)

</details>

#### `purpur.yml`

<details>
<summary><b>use-alternate-keepalive</b> — Recommended value: true</summary>

```yaml
settings:
  use-alternate-keepalive: true
```

Sends one keepalive packet per second to the player. Only ejects the player if none were answered in 30 seconds. Reduces disconnections for players with unstable connections.

> Incompatible with **TCPShield**.

</details>

---

### Chunks

#### `server.properties`

<details>
<summary><b>simulation-distance</b> — Recommended value: 4</summary>

```properties
simulation-distance=4
```

The distance (in chunks) around the player where things actually happen: furnaces, crops, trees, mobs, etc.

Keeping it low (3-4) is intentional. It is complemented by `view-distance` so players see further without the same performance impact.

</details>

<details>
<summary><b>view-distance</b> — Recommended value: 7</summary>

```properties
view-distance=7
```

Chunks sent visually to the client. The total distance will be the larger value between `simulation-distance` and `view-distance`.

> Example: simulation=4, view=7 — the client sees 7 chunks, but only 4 are actively simulated.

</details>

#### `paper-world.yml`

<details>
<summary><b>delay-chunk-unloads-by</b> — Recommended value: 10s</summary>

```yaml
chunks:
  delay-chunk-unloads-by: 10s
```

Keeps chunks loaded for 10 seconds after a player moves away. Prevents constantly loading/unloading the same chunks when a player goes back and forth.

</details>

<details>
<summary><b>max-auto-save-chunks-per-tick</b> — Recommended value: 8</summary>

```yaml
chunks:
  max-auto-save-chunks-per-tick: 8
```

Spreads world saving over time for better average performance. With 20-30+ players, consider increasing to `12` or `16`.

</details>

<details>
<summary><b>prevent-moving-into-unloaded-chunks</b> — Recommended value: true</summary>

```yaml
chunks:
  prevent-moving-into-unloaded-chunks: true
```

Prevents players from entering unloaded chunks, preventing sync loads that clog the main thread.

</details>

<details>
<summary><b>entity-per-chunk-save-limit</b> — See recommended values</summary>

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

Limits how many entities of each type can be saved per chunk. Prevents crashes when loading chunks with absurd amounts of saved projectiles.

</details>

#### `pufferfish.yml`

<details>
<summary><b>max-loads-per-projectile</b> — Recommended value: 8</summary>

```yaml
max-loads-per-projectile: 8
```

Limits how many chunks a projectile can load during its trajectory. Reduces lag from ender pearls and tridents.

</details>

---

### Mobs

#### `bukkit.yml`

<details>
<summary><b>spawn-limits</b> — See recommended values</summary>

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

The formula is: `[active players] x [limit]`. Lower numbers mean less work for the server. Combine with `per-player-mob-spawns: true` in Paper for equitable distribution.

</details>

<details>
<summary><b>ticks-per</b> — See recommended values</summary>

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

Frequency (in ticks) at which the server attempts to spawn each type of entity. Animals and aquatic mobs don't need to spawn every tick, as they don't die as quickly.

</details>

#### `spigot.yml`

<details>
<summary><b>mob-spawn-range</b> — Recommended value: 3</summary>

```yaml
mob-spawn-range: 3
```

Range in chunks where mobs can spawn around the player. Must be less than or equal to simulation-distance and never greater than the despawn range divided by 16.

</details>

<details>
<summary><b>entity-activation-range</b> — See recommended values</summary>

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

Distance from the player at which an entity ticks. Reducing it improves performance but may affect mob farms.

</details>

<details>
<summary><b>entity-tracking-range</b> — See recommended values</summary>

```yaml
entity-tracking-range:
  players: 48
  animals: 48
  monsters: 48
  misc: 32
  other: 64
```

Distance in blocks at which entities are visible to clients. Must be greater than `entity-activation-range`.

</details>

<details>
<summary><b>nerf-spawner-mobs</b> — Recommended value: true</summary>

```yaml
nerf-spawner-mobs: true
```

Mobs spawned by spawners will not have AI. Drastically improves performance on servers with many spawners. Mobs can jump in water if `spawner-nerfed-mobs-should-jump: true` in paper-world.

</details>

#### `paper-world.yml`

<details>
<summary><b>despawn-ranges</b> — See recommended values</summary>

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

- **Soft range:** random probability of despawning
- **Hard range:** instant despawn
- Recommended formula: `(simulation-distance x 16) + 8`

</details>

<details>
<summary><b>per-player-mob-spawns</b> — Recommended value: true</summary>

```yaml
entities:
  spawning:
    per-player-mob-spawns: true
```

Distributes mob spawning equitably among players. Prevents one player with a huge farm from hogging the entire mob cap.

</details>

<details>
<summary><b>max-entity-collisions</b> — Recommended value: 2</summary>

```yaml
entities:
  max-entity-collisions: 2
```

Limits how many collisions an entity can process per tick. `0` makes pushing entities impossible. `2` is sufficient for most cases.

</details>

<details>
<summary><b>update-pathfinding-on-block-update</b> — Recommended value: false</summary>

```yaml
entities:
  update-pathfinding-on-block-update: false
```

Reduces unnecessary pathfinding. Mobs will update their path passively every 5 ticks instead of on every block update.

</details>

#### `pufferfish.yml` — DAB (Dynamic Activation Brain)

> **One of Pufferfish's most important optimizations.**

<details>
<summary><b>dab.enabled</b> — Recommended value: true</summary>

```yaml
dab:
  enabled: true
  max-tick-freq: 20
  activation-dist-mod: 7
```

DAB reduces how much an entity "thinks" based on how far it is from players. Unlike EAR (which makes a hard cutoff), DAB works on a gradient:

- **Nearby entities** — full tick
- **Distant entities** — progressively reduced tick

| Option | Description | Recommended |
|--------|-------------|-------------|
| `max-tick-freq` | Slowest speed for distant entities | `20` |
| `activation-dist-mod` | How close to the player triggers DAB | `7` |

> If DAB breaks your farms, try increasing `activation-dist-mod` or reducing `max-tick-freq`.

</details>

<details>
<summary><b>enable-async-mob-spawning</b> — Recommended value: true</summary>

```yaml
enable-async-mob-spawning: true
```

Moves the computational calculation of mob spawning to a separate thread. Requires `per-player-mob-spawns: true` in Paper. It doesn't spawn mobs asynchronously, only optimizes the pre-calculation.

</details>

#### `purpur.yml`

<details>
<summary><b>villager.lobotomize.enabled</b> — Recommended value: true if villagers cause lag</summary>

```yaml
mobs:
  villager:
    lobotomize:
      enabled: true
      check-interval: 100
```

Villagers that can't pathfind lose their AI and only restock offers periodically. Activate only if villagers are causing lag.

</details>

<details>
<summary><b>entities-can-use-portals</b> — Recommended value: false</summary>

```yaml
settings:
  entities-can-use-portals: false
```

Prevents entities (non-players) from using portals and loading chunks when changing dimensions.

</details>

---

### Miscellaneous

#### `spigot.yml`

<details>
<summary><b>merge-radius</b> — See recommended values</summary>

```yaml
merge-radius:
  item: 3.5
  exp: 4.0
```

Distance at which items and XP orbs merge with each other, reducing entities on the ground. Too high can cause items to "teleport" through walls.

</details>

<details>
<summary><b>hopper-transfer / hopper-check</b> — Recommended value: 8</summary>

```yaml
hopper-transfer: 8
hopper-check: 8
```

Tick delays between hopper actions. Significantly improves performance with many hoppers. Very high values can break hopper clocks.

</details>

#### `paper-world.yml`

<details>
<summary><b>alt-item-despawn-rate</b> — See recommended values</summary>

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

Common and worthless items despawn faster, reducing entities on the ground. An efficient alternative to cleanup plugins.

</details>

<details>
<summary><b>redstone-implementation</b> — Recommended value: ALTERNATE_CURRENT</summary>

```yaml
redstone:
  implementation: ALTERNATE_CURRENT
```

Replaces the vanilla redstone system with a faster implementation based on the [Alternate Current](https://modrinth.com/mod/alternate-current) mod. Reduces redundant block updates.

> May introduce minor inconsistencies with very technical redstone, but the performance gain far outweighs any potential issues.

</details>

<details>
<summary><b>optimize-explosions</b> — Recommended value: true</summary>

```yaml
misc:
  optimize-explosions: true
```

Replaces the vanilla explosion algorithm with a faster one with minimal difference in damage calculation.

</details>

<details>
<summary><b>treasure-maps.enabled</b> — Recommended value: false</summary>

```yaml
environment:
  treasure-maps:
    enabled: false
    find-already-discovered:
      loot-tables: true
      villager-trade: true
```

Generating treasure maps is extremely expensive and can hang the server if the structure is in ungenerated chunks. Only enable if you have pre-generated the world completely.

</details>

<details>
<summary><b>non-player-arrow-despawn-rate / creative-arrow-despawn-rate</b> — Recommended value: 20</summary>

```yaml
entities:
  spawning:
    non-player-arrow-despawn-rate: 20
    creative-arrow-despawn-rate: 20
```

Arrows from mobs and creative mode disappear in 1 second (20 ticks) after impact. Players can't pick them up anyway.

</details>

---

### Helpers and Security

#### `paper-world.yml`

<details>
<summary><b>anti-xray.enabled</b> — Recommended value: true</summary>

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

Much more efficient than any anti-xray plugin. The performance impact is minimal.

- **Mode 1:** Hides specific ores
- **Mode 2 (Recommended):** Replaces blocks with fake ores, more secure but slightly more CPU usage

</details>

<details>
<summary><b>nether-ceiling-void-damage-height</b> — Recommended value: 127</summary>

```yaml
environment:
  nether-ceiling-void-damage-height: 127
```

Void damage above the Nether ceiling (height 128). Prevents players from exploiting the Nether roof. If you modify the Nether height: `[your_nether_height] - 1`.

</details>

---

## Java Startup Flags

> **Minecraft 1.20.4+ requires Java 21.**
> Recommended providers: [Adoptium](https://adoptium.net/) and [Amazon Corretto](https://aws.amazon.com/corretto/)
> Don't use Oracle JDK — licensing changes in 2023 make it unsuitable for production.

### G1GC Recommended for Most

G1GC remains the best choice for most servers in 2026. Aikar's flags are the industry standard:

```bash
java -Xms8G -Xmx8G -XX:+UseG1GC -XX:+ParallelRefProcEnabled -XX:MaxGCPauseMillis=200 -XX:+UnlockExperimentalVMOptions -XX:+DisableExplicitGC -XX:+AlwaysPreTouch -XX:G1NewSizePercent=30 -XX:G1MaxNewSizePercent=40 -XX:G1HeapRegionSize=8M -XX:G1ReservePercent=20 -XX:G1HeapWastePercent=5 -XX:G1MixedGCCountTarget=4 -XX:InitiatingHeapOccupancyPercent=15 -XX:G1MixedGCLiveThresholdPercent=90 -XX:G1RSetUpdatingPauseTimePercent=5 -XX:SurvivorRatio=32 -XX:+PerfDisableSharedMem -XX:MaxTenuringThreshold=1 -Dusing.aikars.flags=https://mcflags.emc.gs -Daikars.new.flags=true --add-modules=jdk.incubator.vector -jar server.jar --nogui
```

> Set `-Xms` and `-Xmx` to the same value to prevent the JVM from resizing the heap during gameplay.

### ZGC for High-Traffic Servers

Generational ZGC has been available since Java 21 and is ideal for servers with 150+ players or high memory allocation rates:

```bash
java -Xms10G -Xmx10G -XX:+UseZGC -XX:+ZGenerational -XX:+AlwaysPreTouch -XX:+DisableExplicitGC -XX:+UnlockExperimentalVMOptions --add-modules=jdk.incubator.vector -jar server.jar --nogui
```

> Don't mix G1GC and ZGC flags. They are completely different collectors. ZGC has sub-millisecond pauses but higher CPU overhead. Use `/spark gcmonitor` to measure before switching.

### G1GC vs ZGC Comparison 2026

| Metric | G1GC | Generational ZGC |
|---------|------|-----------------|
| GC Pauses | 50-200ms | less than 1ms |
| Throughput | High | Moderate |
| CPU Usage | Moderate | Higher |
| Minimum RAM | 4GB | 8GB+ |
| When to Use | Most servers | 150+ players, GC lag issues |

### Automatic Flag Generator

Use [flags.sh](https://flags.sh) to generate the correct flags based on your RAM configuration and server type.

### Bonus SIMD Flag for Pufferfish

```bash
--add-modules=jdk.incubator.vector
```

Add this flag **before** `-jar`. Allows Pufferfish to use SIMD instructions on your CPU, making map rendering in plugins up to 8x faster.

---

## Plugins Myths and Realities

### Plugins to Avoid

#### Item Cleanup Plugins

Completely unnecessary. Replace them with:

- [`merge-radius`](#merge-radius) in spigot.yml
- [`alt-item-despawn-rate`](#alt-item-despawn-rate) in paper-world.yml

Cleanup plugins constantly scan the world, consuming more resources than simply not generating excessive items.

#### Mob Stacker Plugins

Stacking naturally spawned entities causes more lag than it prevents. The server constantly tries to spawn more mobs to replace the stacks. The only acceptable case is with spawners with massive quantities on specialized servers.

#### Plugins That Enable or Disable Other Plugins

Extremely dangerous. Loading a plugin at runtime can cause fatal data errors. Disabling one can break dependencies.

> This includes the `/reload` command. [Read why it's a bad idea](https://madelinemiller.dev/blog/problem-with-reload/).

#### EssentialsX and Similar All-in-One Plugins

EssentialsX loads dozens of features you probably don't use, many deprecated. Look for specialized plugins for the functions you actually need on [SpigotMC](https://www.spigotmc.org/resources/) or [Modrinth](https://modrinth.com/plugins).

#### Anti-Lag and Magic Optimizer Plugins

`LagRemover`, `UltimateLagFixer`, and similar are, at best, useless and, at worst, harmful. All the adjustments they make can be done manually with better control.

### Recommended Plugins

| Plugin | Purpose | Link |
|--------|-----------|--------|
| **Spark** | Real-time CPU and memory profiling | [spark.lucko.me](https://spark.lucko.me) |
| **Chunky** | Efficient chunk pre-generation | [modrinth.com/plugin/chunky](https://modrinth.com/plugin/chunky) |
| **Plan** | Performance and player statistics | [planplugin.com](https://planplugin.com) |
| **FarmControl** | Limit entities in farms | [modrinth.com](https://modrinth.com) |

---

## Measuring Performance

### MSPT — The Basic Metric

```
/mspt
```

Paper's `/mspt` command shows how long the server took to calculate the last few ticks.

| MSPT Value | Status |
|-----------|--------|
| less than 50ms (both first values) | Healthy server (20 TPS) |
| 50-100ms | Under 20 TPS, investigate |
| more than 100ms | Server with serious lag |

The **third value** is the maximum peak — occasional high values are normal.

### Spark — The Essential Profiler

[Spark](https://spark.lucko.me/) is the most important tool for diagnosing lag in depth.

```bash
/spark profiler start
/spark profiler stop
/spark gcmonitor
/spark health
```

Spark shows exactly what code is consuming the most CPU, allowing you to identify the plugin or system responsible for the lag.

> [Complete guide to finding lag spikes with Spark](https://spark.lucko.me/docs/guides/Finding-lag-spikes)

### Timings (Legacy)

```bash
/timings paste
```

The basic built-in diagnostic tool. Has a serious impact on server performance during measurement. Use Spark whenever possible.

If you're using Purpur or Pufferfish, you can disable Timings completely:

```yaml
# paper-global.yml
timings:
  enabled: false
```

### Interpreting the Results

```
TPS = Ticks Per Second (maximum 20)
MSPT = Milliseconds Per Tick (target: less than 50ms)

TPS = 20 when 20 x MSPT <= 1000ms
```

If your average MSPT is close to 50ms, the server is at its limit. Identify the culprit:

1. **Too many mobs** — Reduce spawn-limits, enable DAB
2. **Chunks** — Reduce view and simulation distance
3. **Plugins** — Use Spark to identify the culprit
4. **Redstone** — Enable ALTERNATE_CURRENT
5. **Hopper lag** — Increase hopper-transfer and hopper-check

---

## Exploits and How to Fix Them

Exploits can cause devastating lag spikes or even crash the server. Check the complete guide:

**[Minecraft Exploits and How to Fix Them](https://github.com/zMarkitos/minecraft-exploits-and-how-to-fix-them)**

### The Most Common in 2026

| Exploit | Cause | Solution |
|---------|-------|----------|
| **TNT Dupers** | Thousands of TNT entities | `block-explosion-drop-decay: true` |
| **Carpet Bombing** | Placing and breaking blocks quickly | Block rate limiting |
| **0-Tick Farms** | Bugs in block updates | Paper fixes them automatically |
| **AFK Fish Farms** | Lag from fish quantity | Limit arrow and fish despawning |
| **End Crystal Lag** | End Crystals in a loop | Limit `entity-per-chunk-save-limit` |
| **Treasure Map Lag** | Search in ungenerated chunks | `treasure-maps.enabled: false` |
| **Nether Roof Exploit** | Players above the roof | `nether-ceiling-void-damage-height: 127` |

---

## FAQ

<details>
<summary><b>How much RAM does my server need?</b></summary>

| Players | Recommended RAM |
|-----------|----------------|
| 1-10 | 4-6 GB |
| 10-30 | 6-8 GB |
| 30-60 | 8-12 GB |
| 60+ | 12-16 GB and consider Folia |

Don't allocate more than 12-16GB without measuring first. G1GC scales well up to about 12GB.

</details>

<details>
<summary><b>What's the difference between TPS and MSPT?</b></summary>

- **TPS (Ticks Per Second):** How many ticks the server completes per second. Maximum 20. Below 20 there is noticeable lag.
- **MSPT (Milliseconds Per Tick):** How long each tick takes. Desirable maximum: 50ms. If a tick takes more than 50ms, the next one is delayed, lowering TPS.

They are inversely related: high MSPT causes low TPS.

</details>

<details>
<summary><b>Why isn't there a guide for Modded servers?</b></summary>

Modded servers (Forge, Fabric with mods) have their own optimization ecosystem with mods like Starlight, Lithium, FerriteCore, etc. This guide is specialized in the Paper/Bukkit plugin ecosystem. For modded, look for specific guides from [Fabric](https://fabricmc.net) or [NeoForge](https://neoforged.net).

</details>

<details>
<summary><b>Can I use these configurations on 1.19.x?</b></summary>

Most configurations apply to 1.19 - 1.21.x, although some specific values may vary. Always check the official documentation of the software you're using.

</details>

---

## Contributing

Found incorrect information or want to add something?

1. **Fork** the repository
2. Create a branch: `git checkout -b improvement/my-contribution`
3. Make your changes and commit: `git commit -m 'Add: description of change'`
4. Push: `git push origin improvement/my-contribution`
5. Open a **Pull Request**

You can also open an **Issue** if you find errors or have suggestions.

---

## Sources

| Source | Description |
|--------|-------------|
| [zMarkitos_/zMOptimization_Minecraft_2026](https://github.com/zMarkitos_/zMOptimization_Minecraft_2026) | Original base guide |
| [PaperMC Documentation](https://docs.papermc.io) | Official Paper documentation |
| [Pufferfish Optimization Guide](https://docs.pufferfish.host/optimization/) | Official Pufferfish guide |
| [Purpur Documentation](https://purpurmc.org/docs/) | Official Purpur documentation |
| [Aikar's JVM Flags](https://docs.papermc.io/paper/aikars-flags) | Industry standard JVM flags |
| [flags.sh](https://flags.sh) | Startup flag generator |
| [Spark](https://spark.lucko.me/docs/) | Profiler documentation |
| [Obydux/Minecraft-startup-flags](https://github.com/Obydux/Minecraft-startup-flags) | Modern JVM flag benchmarks |

---

<div align="center">

Made for the Minecraft community

[![GitHub](https://img.shields.io/badge/GitHub-zMarkitos-black?style=for-the-badge&logo=github)](https://github.com/zMarkitos)

<br>

</div>
```
