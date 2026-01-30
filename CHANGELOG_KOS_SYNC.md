# Spy Addon - KOS Sync Feature Changelog

## Version 1.0.17 - Per-Character Stats Tracking

### Changes
- **Per-character stats tracking** - Each of your alts' kills/deaths tracked separately
- **Individual character display** - ALL characters (yours and guildies) shown individually
- **AccountID system** - Each account generates a unique ID to identify alts
- **No duplication** - Guild totals sum individual contributions without double-counting

### How It Works Now
- **Your characters**: Each shown individually (e.g., "Alt A (You): 2-0", "Alt B (Your alt): 1-1")
- **Guild member characters**: Each shown individually (e.g., "Walrus: 2-0", "WalrusAlt (alt): 1-1")
- **Guild Total**: Sum of all individual contributions, grouped by account to prevent double-counting

### Technical Details
- `SpyDB.AccountID` - Unique hex string per account, shared across all characters
- `SpyDB.AccountStats[realm][faction][enemy][charName]` - Per-character PvP stats
- `GetAccountStatsBreakdown(enemy)` - Returns per-character stats table
- `GetAccountStats(enemy)` - Returns combined total (sum of all your chars)
- PvP broadcasts include AccountID for alt identification
- Each character broadcasts their individual stats only

---

## Overview

This update adds a comprehensive **Kill On Sight (KOS) synchronization system** that allows guild members to share KOS data including reasons, attribution (who added whom), and PvP statistics.

───────────────────────────────────

## New Features

### 1. Guild KOS Sync System
> - **Automatic sync on login** - When you log in, the addon requests KOS data from online guildmates
> - **Real-time broadcasts** - When you add/remove KOS players or add reasons, it's instantly shared with the guild
> - **Per-guild-member tracking** - Tracks who in your guild added each KOS player and when

### 2. KOS Reason Attribution
> - Each KOS reason now tracks **who added it** and **when**
> - Reasons display "(added by PlayerName)" in tooltips and the stats window
> - Attribution is preserved when syncing through multiple people (A→B→C chain)

### 3. Guild PvP Statistics
> - Tracks **wins/losses per guild member** against each KOS player
> - Real-time broadcast when you kill or die to a KOS player
> - View aggregated guild stats in the new Guild Details popup

### 4. Guild Details Popup
> **Left-click** any KOS player in the Statistics window to view:
> - Who added this player to KOS (with timestamps)
> - All KOS reasons with attribution
> - Per-guild-member PvP records
> - Guild total win/loss record
> - Your personal record

### 5. Manual Sync & Force Sync Buttons
> - **Sync KOS** button - requests delta sync from guild
> - **Force Sync** button - clears all sync history and requests ALL data from guild
> - Force sync is bidirectional: both requester and responders exchange full data
> - `/spy sync` slash command to manually request sync

### 6. Player Info & Last Seen Time Sync
> - Syncs **class, level, race, guild** info for KOS players
> - Syncs **last seen time** - shows when guildmates last encountered the player
> - Only updates if synced data is more complete or more recent

### 7. Auto-Refresh Statistics Window
> - Statistics window automatically refreshes when receiving sync data
> - No need to close/reopen the window to see new entries

### 8. New Settings
> - **"Share KOS reasons with guild"** - Enable/disable the sync feature
> - **"Show KOS sync notifications"** - Toggle chat notifications for sync events
> - **"Enable KOS sync debug messages"** - Toggle detailed debug output for troubleshooting sync issues

───────────────────────────────────

## Files Changed

### Spy.lua
```
• Added KOS sync message handler (HandleKOSSyncMessage)
• Added sync protocol functions:
  - RequestKOSSync(forceFullSync) - Request sync on login (with optional FORCE flag)
  - ManualKOSSync() - Manual sync command
  - ForceFullKOSSync() - Force full sync (clears sync times, sends FORCE flag)
  - SendKOSSyncResponse() - Send data to requesting player
  - ProcessKOSSyncData() - Process received sync data (includes auto-refresh)
  - SerializeKOSSyncData() / SendKOSSyncChunks() - Data serialization
• Added real-time broadcast receivers:
  - ReceiveKOSAdd() - Handle KOS add notifications
  - ReceiveKOSReason() - Handle reason add notifications
  - ReceiveKOSRemoval() - Handle removal notifications
  - ReceiveKOSPvPUpdate() - Handle PvP stat updates
• Added BroadcastKOSPvPUpdate() - Broadcast your kills/deaths
• Added MigrateKOSReasons() - Migrate old reason format to new attributed format
• Added EscapeSyncString() / UnescapeSyncString() - Protocol escaping
• Added new profile settings: ShareKOSReasons, KOSSyncNotifications, KOSSyncDebug
• Added data structures: KOSSyncTimes, KOSRemovals
• Modified PlayerEnteringWorldEvent() to trigger sync on login
• Modified kill/death tracking to broadcast PvP updates for KOS players
```

### List.lua
```
• Modified AddKOSData():
  - Initialize kosAddedBy tracking
  - Broadcast KOSR|ADD to guild
• Added BroadcastKOSAdd() - Send KOS add to guild
• Modified RemoveKOSData():
  - Broadcast KOSR|REM to guild
• Added BroadcastKOSRemoval() - Send KOS removal to guild
• Modified SetKOSReason():
  - New format with {text, addedBy, addedAt, source} structure
  - Broadcast KOSR|RSN to guild
• Added BroadcastKOSReason() - Send reason to guild
• Modified RegenerateKOSListFromCentral():
  - Added kosAddedBy merge for cross-character sharing
  - Added guildStats merge for cross-character sharing
```

### MainWindow.lua
```
• Modified KOS reason checkbox logic to handle new table format
• Modified KOS tooltip to show reason attribution "(added by PlayerName)"
```

### SpyStats.lua
```
• Added Sync button initialization
• Modified reason display to handle new attributed format
• Added "Guild Details" option to right-click menu
• Added left-click handler to show Guild Details popup
• Added CreateGuildDetailsPopup() - Create the popup frame
• Added ShowGuildDetailsPopup() - Display detailed KOS info
```

### SpyStats.xml
```
• Added "Sync KOS" button next to Refresh button
• Added "Force Sync" button for full data sync
```

### Locales/Spy-enUS.lua
```
Added new localization strings for:
• Settings labels and descriptions
• Chat notification messages
• Guild Details popup labels
• Sync status messages
```

───────────────────────────────────

## Data Structures

### New: `playerData.kosAddedBy`
```lua
kosAddedBy = {
    ["GuildMember1"] = 1706000000,  -- timestamp when they added
    ["GuildMember2"] = 1706100000,
    ["YourName"] = 1706050000
}
```

### New: `playerData.guildStats`
```lua
guildStats = {
    ["GuildMember1"] = {
        wins = 5,
        losses = 2,
        lastUpdate = 1706000000
    }
}
```

### Modified: `playerData.reason`
**Old format:**
```lua
reason = {
    ["Camped me"] = true,
    ["Other"] = "Custom reason text"
}
```

**New format:**
```lua
reason = {
    ["Camped me"] = {
        addedBy = "PlayerName",
        addedAt = 1706000000,
        source = "local"  -- or "sync"
    },
    ["Other"] = {
        text = "Custom reason text",
        addedBy = "PlayerName",
        addedAt = 1706000000,
        source = "local"
    }
}
```

───────────────────────────────────

## Communication Protocol

All messages use prefix `KOSR|` via AceComm on GUILD channel.

```
KOSR|ADD  →  EnemyName|Timestamp|ReasonKey|ReasonText     →  KOS player added
KOSR|RSN  →  EnemyName|ReasonKey|ReasonText|Timestamp     →  Reason added
KOSR|REM  →  EnemyName|Timestamp                          →  KOS player removed
KOSR|REQ  →  SenderCharName|FORCE                         →  Request sync (FORCE optional)
KOSR|RSP  →  SerializedData                               →  Sync response
KOSR|PVP  →  EnemyName|WIN/LOSS|Wins|Losses|Timestamp     →  PvP update
```

### Sync Data Format
```
name:addedTime;R~reasonKey~addedBy~addedAt~text;G~member~wins~losses~lastUpdate;A~adder~timestamp;P~class~level~race~guild~lastSeen;W~wins~loses,...
```
> - `R~` prefix = Reason data
> - `G~` prefix = Guild stats data
> - `A~` prefix = KOS adder data
> - `P~` prefix = Player info (class, level, race, guild, lastSeen)
> - `W~` prefix = Sender's personal wins/loses against this enemy

───────────────────────────────────

## Edge Cases Handled

1. **A→B→C attribution chain** - Original adder is preserved through multiple syncs
2. **Offline users** - `KOSSyncTimes` tracks per-person sync times; full sync on reconnect
3. **Multiple adders** - `kosAddedBy` is a map, keeps all adders with earliest timestamps
4. **Cross-character sharing** - `kosAddedBy` and `guildStats` now properly merge between alts
5. **Legacy data migration** - Old reason format automatically migrated to new attributed format
6. **New adders while offline** - `kosAddedBy` timestamps checked in sync include logic
7. **Bidirectional force sync** - FORCE flag tells responders to send ALL data, not just deltas
8. **Player info preservation** - Only overwrites class/level/race/guild if local data is missing
9. **Last seen time** - Only updates if synced time is more recent than local time
10. **UI auto-refresh** - Stats window refreshes automatically when sync data is received

───────────────────────────────────

## Usage

1. Enable **"Share KOS reasons with guild"** in Spy Data Management settings
2. Add players to KOS as normal - data syncs automatically
3. Click KOS players in Statistics window to see guild details
4. Use **"Sync KOS"** button or `/spy sync` to request delta sync (changes since last sync)
5. Use **"Force Sync"** button to request ALL data from guildmates (useful for first-time setup or troubleshooting)
6. Enable **"Enable KOS sync debug messages"** to troubleshoot sync issues
