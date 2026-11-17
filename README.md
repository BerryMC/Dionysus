# Dionysus

[![GitHub all releases](https://img.shields.io/github/downloads/BerryMC/Dionysus/total)](https://github.com/BerryMC/Dionysus/releases)
[![GitHub contributors](https://img.shields.io/github/contributors/BerryMC/Dionysus)](https://github.com/BerryMC/Dionysus/graphs/contributors)
[![GitHub issues](https://img.shields.io/github/issues/BerryMC/Dionysus)](https://github.com/BerryMC/Dionysus/issues)

###### A fork of [nopjmp/Dionysus](https://github.com/nopjmp/Dionysus), maintained by [BerryMC](https://github.com/BerryMC).

----

#### 1.12.2 [Paper](https://github.com/PaperMC/Paper) fork aimed at improving server performance for anarchy servers.
###### Note: This fork is based off [Aikar's EMC Framework](https://github.com/starlis/empirecraft)

----

## Server Admins

Dionysus uses the same paperclip jar system that Paper uses.
You can download the latest release of Dionysus [here](https://github.com/BerryMC/Dionysus/releases/latest).
You can also [build it yourself](https://github.com/BerryMC/Dionysus#building).

### Configuration

All server yaml configs live under the `configs/` directory: `bukkit.yml`, `spigot.yml`, `paper.yml`, `commands.yml`, `help.yml` and `dionysus.yml`. Legacy root-level files are migrated automatically on first start. `server.properties`, `eula.txt` and the JSON data files (ops, whitelist, bans, usercache) stay in the server root.

Notable `configs/dionysus.yml` options (everything is optional and keeps vanilla behavior unless stated otherwise):

| Option | Default | Description |
|---|---|---|
| `crop.optimize-growth` | `false` | Single-RNG-call crop/stem growth rolls |
| `pathfinding.optimize` | `false` | Skip A* pathfinding when no player is nearby (`pathfinding.max-player-distance`) |
| `scheduled-ticks.optimize` | `false` | Faster scheduled tick (NextTickListEntry) processing loop |
| `hopper.optimize` | `false` | Skip hopper suction entity scans when provably empty (minecart-safe) |
| `async-json-list-save` | `true` | Save ops/whitelist/ban JSON lists asynchronously |
| `egg-can-knockback` | `true` | Thrown eggs knock back players (changes vanilla combat) |
| `use-io-uring` | `false` | Experimental io_uring transport — do not enable unless you know what it is |

JVM flags: `-DDionysus.enableFMA=true` accelerates entity distance math with fused multiply-add; `--add-modules=jdk.incubator.vector` enables the SIMD optimizations on Java 17/18.

## Contributing

*Anyone can contribute. Just follow the below steps!*

1. [Fork Dionysus](https://github.com/BerryMC/Dionysus/fork)
2. [Build Dionysus](https://github.com/BerryMC/Dionysus#building)
3. [Create Pull Request](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/creating-a-pull-request-from-a-fork)

## Plugin Developers
###### *With each Dionysus update you must update your dependency.*

In order to use Dionysus as a dependency you must [build it yourself](https://github.com/BerryMC/Dionysus#building)
This will install it into your local maven repository (`~/.m2/repository`). Then add the following to your `pom.xml`:

#### Dionysus-API Maven Dependency:
```xml
<dependency>
    <groupId>dev.pomf.dionysus</groupId>
    <artifactId>dionysus-api</artifactId>
    <version>1.12.2-R0.1-SNAPSHOT</version>
    <scope>provided</scope>
</dependency>
```
#### Dionysus-Server Maven Dependency:
```xml
<dependency>
    <groupId>dev.pomf.dionysus</groupId>
    <artifactId>dionysus</artifactId>
    <version>1.12.2-R0.1-SNAPSHOT</version>
    <scope>provided</scope>
</dependency>
```

## Building

**Requirements:**

- You need `git` installed, with a configured user name and email.
  On windows you need to run from git bash.
- You need `maven` installed
- You need `jdk 1.8` to decompile and `jdk` 17+ installed to compile (and `jre` 17+ to run)
- Anything else that `paper` requires to build

**If all you want is a paperclip server jar run `./dionysus jar`**

#### Setting up `Dionysus-API` and `Dionysus-Server`

1. Run `./dionysus patch` in your project root.
2. Run `./dionysus build` to build the respective api and server jars.

#### Creating a patch

Patches are effectively just commits in either `Dionysus-API` or `Dionysus-Server`.

1. Create commit in `Dionysus-API` or `Dionysus-Server`.
2. Run `./dionysus rb` in your project root.
3. Commit patch to `Dionysus` repo.

*Modifying commits will also modify its corresponding patch file.*

### License

The PATCHES-LICENSE file describes the license for api & server patches,
found in `./patches` and its subdirectories except when noted otherwise.

Everything else is licensed under the MIT license, except when note otherwise.
See [empirecraft](https://github.com/starlis/empirecraft) and [byof](https://github.com/electronicboy/byof)
for the license of material used/modified by this project.
