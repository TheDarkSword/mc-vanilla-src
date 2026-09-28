# mc-vanilla-src

Get the full, readable and **recompilable** source code of Minecraft: Java Edition (client + server) with one command, and move your own changes to the next Minecraft version with a second one.

- Always on the latest release: a bot checks for new versions every day at 00:00 UTC and bumps this repository by itself.
- Works on Linux, macOS and Windows.
- Original Mojang names everywhere (since 26.1 Minecraft is no longer obfuscated, so no mappings are involved).
- Ready for IntelliJ IDEA / Eclipse / VS Code, with all the libraries resolved.
- Recompile, run the client or a server, or build a jar from your modified code.

> [!WARNING]
> This repository contains **only build scripts**. The Minecraft code is generated on your machine and is the property of Mojang Studios / Microsoft.
> **Do not publish the generated `src/` folder** (or anything built from it) anywhere public. Keep it local or in a private repository.

## Requirements

- **Git**
- **Java 17 or newer**, only to start Gradle. The JDK Minecraft needs (currently 25) is downloaded automatically if you do not have it.
- About 6 GB of free RAM and 2 GB of disk space.

## Quick start

```sh
git clone https://github.com/TheDarkSword/mc-vanilla-src.git
cd mc-vanilla-src
```

| Linux / macOS     | Windows          |
|-------------------|------------------|
| `./gradlew setup` | `gradlew setup`  |

The first run takes a few minutes (it downloads and decompiles the game), later runs reuse the cache.
When it finishes you have:

```
src/
├── main/java/        ← all Minecraft code (net.minecraft.*, com.mojang.*)
├── main/resources/   ← everything else from the game jar (assets, data, ...)
└── .mc-version       ← which version this is, e.g. 26.3-1
```

`src/` is a Git repository of its own: branch `main` is where you work, branch `vanilla` always holds the untouched code.

To open it in an IDE, open the `mc-vanilla-src` folder as a Gradle project (IntelliJ IDEA: *File → Open*, pick the folder).

## Commands

On Windows write `gradlew` (cmd) or `.\gradlew` (PowerShell) instead of `./gradlew`. All of them can also be run from your IDE's Gradle panel (group *minecraft*).

| Command                   | What it does                                                                |
|---------------------------|-----------------------------------------------------------------------------|
| `./gradlew setup`         | Decompiles Minecraft into `src/` (first time only).                         |
| `./gradlew build`         | Recompiles `src/` and builds `build/libs/minecraft-<version>.jar`.          |
| `./gradlew runClient`     | Recompiles and starts the game (offline, username `Dev`, files in `run/client`). |
| `./gradlew runServer`     | Recompiles and starts a dedicated server in `run/server`.                   |
| `./gradlew updateSources` | Moves `src/` to the Minecraft version of this checkout, keeping your changes. |

The first `runClient` downloads the game assets (sounds, languages, ...). The first `runServer` stops right away and asks you to accept Mojang's EULA in `run/server/eula.txt`.

## Updating to a new Minecraft version

Say you have been working on 26.3 and 26.4 comes out:

```sh
git pull                    # this repository is now on 26.4
./gradlew updateSources
```

What happens inside `src/`:

```
vanilla:  26.3 ─────────────────── 26.4
           │                         │
main:      └── your changes ──────── merge ── (you keep working here)
                       │
26.3:                  └── frozen copy of your 26.3 code
```

1. Your current work is saved, untouched, in a branch named after the old version (`26.3`).
2. The new code is decompiled and committed on `vanilla`.
3. `vanilla` is merged into `main`. Git carries your changes over and only stops where **Mojang changed the same lines you did**.

If there are conflicts, the task lists them. Fix them in `src/` like any Git merge, then:

```sh
git -C src add -A
git -C src commit
```

(or `git -C src merge --abort` to go back.) Commit your own work before updating: the task refuses to run on uncommitted changes.

## Other versions

This repository follows the same idea: `latest` is always the latest release, and every older version is kept in a branch with its name.

```sh
git checkout 26.2
./gradlew setup
```

Want a snapshot or a specific build? Any [NeoForm version](https://maven.neoforged.net/#/releases/net/neoforged/neoform) works for a single run:

```sh
./gradlew setup -Pneoform_version=26.4-snapshot-1-1
```

## How it works

- [NeoForm](https://github.com/neoforged/NeoForm) (by NeoForged) describes how to decompile each Minecraft version and ships the small patches that make the output compile again.
- [NeoFormRuntime](https://github.com/neoforged/NeoFormRuntime) runs that pipeline: download client + server, merge them, decompile with [Vineflower](https://github.com/Vineflower/vineflower), apply the patches.
- `build.gradle` takes the result, adds the libraries listed in Mojang's version manifest, and wires up the Git branches and the run tasks.
- `.github/workflows/bump.yml` checks NeoForged's Maven every day at 00:00 UTC. When a new release appears, it makes sure it decompiles and recompiles, creates the branch for the old version, and bumps `neoform_version` in `gradle.properties` on `latest`. Snapshots, pre-releases and release candidates are skipped.

## Credits

- **[Hexeption](https://github.com/Hexeption)** and the contributors of **[MCP-Reborn](https://github.com/Hexeption/MCP-Reborn)**, which this project is inspired by. Thank you for keeping Minecraft modding and research accessible for years.
- **[NeoForged](https://github.com/neoforged)** for NeoForm and NeoFormRuntime, which do the actual heavy lifting.
- **[Vineflower](https://github.com/Vineflower/vineflower)**, the decompiler.
- **Mojang Studios**, for making Minecraft and for no longer obfuscating it.

## License

The scripts in this repository are released under the [MIT License](LICENSE).
Minecraft and all code generated from it belong to Mojang Studios / Microsoft and are subject to the [Minecraft EULA](https://www.minecraft.net/eula). This project is not affiliated with or endorsed by Mojang or Microsoft.
