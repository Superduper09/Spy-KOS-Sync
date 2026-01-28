# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Spy is a World of Warcraft addon for detecting and alerting players to the presence of nearby enemy players. It targets WoW Classic/TBC (Interface: 20505, version 2.0.8).

## Technology Stack

- **Language**: Lua (WoW addon API)
- **Framework**: Ace3 library suite (AceAddon, AceDB, AceEvent, AceConsole, AceComm, AceTimer, AceLocale, AceConfig, AceGUI)
- **Dependencies**: LibSharedMedia-3.0, HereBeDragons-2.0 (map pins), LibChatAnims

## Architecture

### Core Files

- **Spy.lua**: Main addon logic, initialization, options configuration, event handling, player detection via combat log parsing
- **SpyData.lua**: Data layer module handling player storage, session management, PvP tracking, and win/loss statistics
- **SpyStats.lua**: Statistics window module for displaying enemy encounter history with sorting/filtering
- **MainWindow.lua**: Main UI window creation, bar management, tooltips, alert window, map pins
- **List.lua**: Large file containing list management logic (Nearby, Last Hour, Ignore, Kill On Sight lists)
- **Widgets.lua**: Base frame creation utilities
- **Colors.lua**: Color management and registration system
- **Fonts.lua**: Font string management
- **WindowOrder.lua**: Window z-order management

### Data Storage

- **SpyDB**: Global saved variables (cross-character data including KoS data per realm/faction)
- **SpyPerCharDB**: Per-character saved variables containing:
  - `PlayerData`: Enemy player information (name, level, class, race, guild, wins, loses, timestamps, location)
  - `KOSData`: Kill on Sight list
  - `IgnoreData`: Ignored players list

### Key Systems

1. **Detection System**: Parses combat log events (`COMBAT_LOG_EVENT_UNFILTERED`) to detect enemy players by their actions
2. **Alert System**: Visual/audio alerts for detected enemies, stealth detection, and KoS players
3. **List Management**: Four lists (Nearby, Last Hour, Ignore, Kill On Sight) with automatic expiration
4. **Communication**: Shares detected players with party/raid/guild via AceComm
5. **Map Integration**: Shows detected players on world map and minimap using HereBeDragons pins

### Localization

Localization files in `Locales/` directory (enUS, deDE, esES, esMX, frFR, itIT, koKR, ptBR, ruRU, zhTW, zhCN). The `L` table from AceLocale is used throughout for all user-facing strings.

## Development Notes

### Addon Load Order (from Spy.toc)

1. Embeds.xml (loads Libs/)
2. Spy.xml (main XML templates)
3. Locale files
4. Core Lua files in dependency order

### Common Patterns

- Modules created via `Spy:NewModule("ModuleName", mixins...)`
- Options use AceConfig declarative table structure in `Spy.options`
- UI elements use WoW frame API with BackdropTemplate
- Color/font registration via `Spy.Colors:Register*` methods

### Slash Commands

- `/spy` - Show command list
- `/spy enable` - Enable and show window
- `/spy show` - Show window
- `/spy reset` - Reset window position
- `/spy clear` - Clear nearby list
- `/spy config` - Open options
- `/spy ignore <name>` - Toggle ignore
- `/spy stats` - Open statistics window
