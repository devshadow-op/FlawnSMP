# FlawnSMP

A companion plugin for the FlawnSMP Paper server (Minecraft 1.21.11). It adds a server menu,
a safe `/home` teleport (countdown + cancel-on-move/damage + combat lock), new `/hearts
give|set|take` admin commands, and an in-game Admin Panel — all built to sit on top of the
server's existing **LifeStealZ** (hearts) and **EssentialsX** (`/home`) plugins rather than
replacing them.

## Requirements

- Paper (or a Paper fork) for Minecraft **1.21.11**
- Java 21
- [LifeStealZ](https://modrinth.com/plugin/lifestealz) - for hearts
- [PlaceholderAPI](https://www.spigotmc.org/resources/placeholderapi.6245/) - required so FlawnSMP
  can *read* live heart values from LifeStealZ (LifeStealZ has no other public API)
- [EssentialsX](https://essentialsx.net/) - for `/home`, `/sethome`, `/delhome`

FlawnSMP will still start up without any of these installed - it just shows an "unavailable"
message wherever that integration is needed, instead of crashing or erroring.

## Building

This project uses Maven. From the project root:

```bash
mvn clean package
```

The compiled plugin will be at `target/FlawnSMP-1.0.0.jar`.

(Building requires internet access the first time, so Maven can download the Paper API and
PlaceholderAPI API jars it compiles against - both are `provided` scope, so they are **not**
bundled into the final jar.)

## Installing

1. Make sure LifeStealZ, PlaceholderAPI and EssentialsX are already installed and working.
2. Drop `FlawnSMP-1.0.0.jar` into your server's `plugins/` folder.
3. Restart (or `/reload`, though a restart is safer) the server.
4. A default `plugins/FlawnSMP/config.yml` will be generated - you generally never need to
   touch it by hand, since every setting in it is editable in-game via `/flawnsmp admin`.

## Commands & Permissions

| Command | Description | Permission | Default |
|---|---|---|---|
| `/flawnsmp` | Opens the FlawnSMP menu | `flawnsmp.use` | everyone |
| `/flawnsmp admin` | Opens the Admin Panel | `flawnsmp.admin` | op |
| `/hearts` | Shows your own hearts | `flawnsmp.hearts` | everyone |
| `/hearts <player>` | Shows another player's hearts | `flawnsmp.hearts` | everyone |
| `/hearts give <player> <amount>` | Gives hearts | `flawnsmp.hearts.give` | op |
| `/hearts set <player> <amount>` | Sets hearts to an exact value | `flawnsmp.hearts.set` | op |
| `/hearts take <player> <amount>` | Takes hearts away | `flawnsmp.hearts.take` | op |

## How the integrations work (worth understanding before you demo this)

- **Hearts (LifeStealZ):** FlawnSMP never stores or calculates hearts itself. It *reads* the
  current value using LifeStealZ's own PlaceholderAPI placeholders (`%lifestealz_hearts%`,
  `%lifestealz_maxhearts%`), and *writes* changes by running LifeStealZ's own admin command
  (`/lifestealz hearts add|set <player> <amount>`) from the console. This means FlawnSMP can
  never desync from LifeStealZ's real data.
- **Homes (EssentialsX):** the Home menu button and the `/home`-equivalent behaviour just run
  Essentials' own `/home`, `/sethome` and `/delhome` commands as the player. FlawnSMP adds a
  countdown, movement/damage cancellation and a combat lock *on top* of that - it does not
  store home locations itself.
- **If either plugin is missing:** the relevant menu/command shows a clear "unavailable"
  message instead of throwing an error. Check `/flawnsmp admin` → Server Information for a
  live status view of both integrations.

## Project structure

```
com.flawnsmp
├── FlawnSMP.java                 - main plugin class
├── commands/                     - /flawnsmp and /hearts
├── config/ConfigManager.java     - reads/writes config.yml (backs the Admin Panel)
├── data/PlayerDataManager.java   - kills/deaths/playtime (LifeStealZ doesn't track these)
├── gui/                          - the player-facing FlawnSMP menu
│   └── admin/                    - the OP-only Admin Panel
├── integration/                  - LifeStealZ + Essentials integration layers
├── listeners/                    - inventory clicks, join/quit, combat, chat-input capture
└── teleport/HomeTeleportManager.java - countdown / cancel-on-move / combat lock
```

## Note for the classroom

This plugin was written from a Dutch prompt that specified server details (IP, Discord invite,
LifeStealZ + EssentialsX as the existing systems) that were confirmed before writing any code.
That confirmation step matters: without knowing *which* Lifesteal/homes plugin was already
running, the integration code would have had to guess or use fake placeholder logic instead of
real API calls.
