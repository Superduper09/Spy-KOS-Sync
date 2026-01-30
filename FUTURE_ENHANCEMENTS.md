# Spy Addon - Future Enhancements

This file tracks potential improvements to investigate later.

---

## 1. Combat Lockdown UI Delay Fix

**Issue**: When detecting players during combat, the sound alert plays immediately but player names don't appear in the Spy window until combat ends. The header count ("Nearby 2") updates correctly, but the actual name bars are delayed.

**Root Cause**: `ManageBarsDisplayed()` in `MainWindow.lua:933-941` uses `row:Show()` and `row:Hide()` which are protected functions blocked by WoW's combat lockdown.

**Proposed Solution**: Replace Show()/Hide() with SetAlpha()/EnableMouse() which are not protected:

```lua
-- Instead of:
row:Show()  -- blocked in combat
row:Hide()  -- blocked in combat

-- Use:
row:SetAlpha(1)
row:EnableMouse(true)

row:SetAlpha(0)
row:EnableMouse(false)
```

**Files to Modify**:
- `MainWindow.lua` - `ManageBarsDisplayed()` function (~line 922)
- Possibly row creation code to ensure rows start as shown but alpha=0

**Considerations**:
- Need to pair SetAlpha with EnableMouse to prevent invisible rows from capturing mouse events
- Rows are SecureActionButtonTemplate frames (click-to-target), so invisible clickable buttons could cause issues without EnableMouse(false)
- Test thoroughly in and out of combat scenarios

**Priority**: Low - Edge case that only affects detection during active combat

---

## 2. Export/Import KOS List

**Feature**: Ability to export and import KOS data for backup and sharing purposes.

**Export Format** (CSV or text):
```
PlayerName,AddedTimestamp,Class,Level,Race,Guild,Wins,Losses,Reasons,GuildStats
"Ganker",1706000000,"ROGUE",60,"Undead","Bad Guild",5,2,"Camped me|Corpse camped","Member1:3:1|Member2:2:0"
```

**Data to Include**:
- Player name
- When added to KOS
- Player info (class, level, race, guild)
- Personal wins/losses
- All KOS reasons (with attribution if available)
- Guild stats (per-member wins/losses)
- `kosAddedBy` data (who in guild added this player)

**Import Behavior**:
- Add new entries that don't exist locally
- Option to merge or overwrite existing entries
- Validate format before importing
- Show preview/summary before confirming import

**UI Implementation**:
- Add "Export KOS" and "Import KOS" buttons to SpyStats window or settings
- Export creates a text file or copies to clipboard
- Import via file selection or paste from clipboard

**Use Cases**:
- Backup KOS data before major changes
- Share KOS list with friends outside guild
- Transfer data between accounts
- Recovery if SavedVariables get corrupted

**Files to Modify**:
- `SpyStats.lua` or `Spy.lua` - Export/Import functions
- `SpyStats.xml` - UI buttons
- `Locales/Spy-enUS.lua` - New strings

**Priority**: Medium - Useful for data management and backup

---

## 3. Same Faction Watch List (Friendly Player Tracking)

**Feature**: Track and alert for same-faction players (e.g., toxic players, ninja looters) similar to how KOS tracks enemy players. Includes full guild sync support.

**Use Cases**:
- Alert when a known toxic player joins your dungeon group
- Track ninja looters or scammers across the server
- Share "avoid this player" lists with guildmates
- Get notified when encountering problematic players in the world

**Detection Methods**:

1. **Party/Raid Join** (easiest, most useful):
   ```lua
   -- On GROUP_ROSTER_UPDATE event
   for i = 1, GetNumGroupMembers() do
       local name = UnitName("party"..i) or UnitName("raid"..i)
       if WatchListData[name] then
           -- Alert!
       end
   end
   ```

2. **Combat Log** (for world encounters):
   - Same as enemy detection but include friendly faction
   - Filter by `COMBATLOG_OBJECT_REACTION_FRIENDLY`

3. **Target Changed**:
   - Check `PLAYER_TARGET_CHANGED` if target is on watch list

**Data Structure**:
```lua
SpyPerCharDB.WatchListData = {
    ["PlayerName"] = timestamp,  -- when added
}

SpyPerCharDB.PlayerData["PlayerName"] = {
    -- Existing fields work for both factions
    watched = 1,  -- flag similar to kos = 1
    watchAddedBy = { ["GuildMember"] = timestamp },
    -- reason, guildStats, etc. all reusable
}
```

**Guild Sync**:
- Reuse existing sync infrastructure (90% code reuse)
- New message prefix: `WTCH|` (vs `KOSR|` for KOS)
- Same protocol: `WTCH|ADD`, `WTCH|REM`, `WTCH|RSN`, `WTCH|REQ`, `WTCH|RSP`
- Separate sync times: `WatchSyncTimes` (vs `KOSSyncTimes`)

**UI Changes**:
- New "Watch List" tab in SpyStats window (alongside KOS, Ignore)
- Different color scheme to distinguish from KOS (maybe yellow/orange vs red)
- Same right-click menu options (Add Reason, Remove, Guild Details)
- Setting: "Alert when Watch List player joins group"
- Setting: "Share Watch List with guild"

**Alert Differences from KOS**:
- Different sound file (not threatening like KOS)
- Different alert message: "Watch List player detected" vs "Kill On Sight"
- Maybe different color in main window if detected in world

**Files to Modify**:
- `Spy.lua` - New sync handlers (`WTCH|` prefix), new settings
- `SpyData.lua` - WatchListData management
- `List.lua` - Detection hooks, alert functions
- `SpyStats.lua` - Watch List tab, filtering
- `SpyStats.xml` - UI for new tab
- `Locales/Spy-enUS.lua` - New strings

**Existing Addon Comparison**:
- **PlayerNotes** - Local only, no sync, no alerts
- **Global Ignore List** - Cross-character but not cross-player sync
- **No known addon** combines watch list + guild sync + alerts

**Priority**: Medium-High - Unique feature, high reuse of existing sync code

---

## 4. "Added By" Column in Statistics Window

**Feature**: Show who added each KOS player directly in the Statistics window as a sortable column.

**Current Behavior**:
- "Added By" info only visible in Guild Details popup (left-click on KOS player)
- Users must click each entry individually to see attribution

**Proposed UI**:
- New column "Added By" in Statistics list header
- Shows first adder (earliest timestamp) or most recent adder
- Sortable like other columns (Name, Level, Guild, Wins, Losses)
- Truncate long names with "..." if needed

**Data Source**:
```lua
playerData.kosAddedBy = {
    ["GuildMember1"] = 1706000000,  -- timestamp
    ["GuildMember2"] = 1706100000,
}
-- Display: Show member with earliest timestamp (original adder)
```

**Files to Modify**:
- `SpyStats.lua` - Add column header, row display, sort function
- `SpyStats.xml` - Column width adjustments
- `Locales/Spy-enUS.lua` - "Added By" header string

**Priority**: Low - Nice to have, info already available via popup

---

## 5. Statistics Window Column Customization

**Feature**: Allow users to show/hide columns in the Statistics window based on preference.

**Columns to Toggle**:
- Name (always shown)
- Level
- Class
- Race
- Guild
- Wins
- Losses
- W/L (combined wins-losses)
- Added By (if implemented)
- Last Seen

**UI Implementation**:
- Right-click on column header → dropdown menu with checkboxes
- Or: Settings panel with column toggle checkboxes
- Save preferences in Spy.db.profile

**Settings Structure**:
```lua
Spy.db.profile.StatsColumns = {
    showLevel = true,
    showClass = true,
    showRace = false,  -- hidden by default
    showGuild = true,
    showWins = true,
    showLosses = true,
    showAddedBy = false,
    showLastSeen = true,
}
```

**Files to Modify**:
- `SpyStats.lua` - Dynamic column rendering
- `SpyStats.xml` - Flexible column widths
- `Spy.lua` - Default settings
- `Locales/Spy-enUS.lua` - Setting labels

**Priority**: Low - Power user feature

---
