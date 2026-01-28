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
