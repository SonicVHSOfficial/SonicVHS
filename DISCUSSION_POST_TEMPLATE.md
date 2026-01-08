# 💾 How to Implement a Save Select Menu in Orbinaut Framework 2

**Category:** Q&A  
**Tags:** save-system, ui, game-development, tutorial

---

## 🎯 Purpose

This discussion provides a comprehensive guide for implementing a save select menu system in games built with Orbinaut Framework 2. Whether you're creating your first Sonic fangame or adding features to an existing project, this guide covers everything you need to know.

---

## 📋 What You'll Learn

- Basic save slot system implementation
- Data structure and persistence
- UI/UX design for save menus
- Input handling and navigation
- Advanced features (delete, copy, validation)
- Best practices and testing

---

## 🤔 Common Questions

### 1. Why do I need a save select menu?

A save select menu allows players to maintain multiple independent save files, enabling:
- Multiple players to use the same game installation
- Experimentation without losing progress
- Classic Sonic game experience (like Sonic 3 & Knuckles)

### 2. What components do I need?

Essential components include:
- **Save Slot Display** (3-4 slots typically)
- **Slot Information** (progress, character, emeralds)
- **Selection Cursor** (visual feedback)
- **Navigation Controls** (up/down/select)
- **Additional Options** (delete, no-save mode)

---

## 🗂️ Save Data Structure

Here's a recommended structure for each save slot:

```gml
SaveSlot {
    slotNumber: 0-2
    isUsed: boolean
    playerCharacter: character ID
    currentZone: zone ID
    currentAct: act number
    emeraldsCollected: bitmask (0b1111111 for all 7)
    lives: number (3 default)
    continues: number
    totalPlayTime: seconds
    lastPlayed: timestamp
}
```

**Storage Options:**
- Game Maker: `ini_open()` / `ini_write_*()` functions
- File-based: `save0.dat`, `save1.dat`, `save2.dat`
- Location: Game's local data directory

---

## 🎨 Implementation Guide

### Step 1: Create the Menu Scene

```gml
// Create rm_save_select room
// Add obj_save_select_controller
```

### Step 2: Initialize Save Data

```gml
// CREATE EVENT
selectedSlot = 0;
maxSlots = 3;

for (var i = 0; i < maxSlots; i++) {
    saveData[i] = LoadSaveSlot(i);
}
```

### Step 3: Handle Input

```gml
// STEP EVENT
var _up = keyboard_check_pressed(vk_up);
var _down = keyboard_check_pressed(vk_down);
var _select = keyboard_check_pressed(vk_space);

if (_up) {
    selectedSlot = (selectedSlot - 1 + maxSlots) % maxSlots;
    audio_play_sound(snd_menu_move, 1, false);
}

if (_down) {
    selectedSlot = (selectedSlot + 1) % maxSlots;
    audio_play_sound(snd_menu_move, 1, false);
}

if (_select) {
    global.currentSaveSlot = selectedSlot;
    StartGameWithSave(selectedSlot);
}
```

### Step 4: Draw the Menu

```gml
// DRAW EVENT
for (var i = 0; i < maxSlots; i++) {
    var _y = 100 + (i * 60);
    var _col = (i == selectedSlot) ? c_yellow : c_white;
    
    draw_set_color(_col);
    draw_text(100, _y, "SLOT " + string(i + 1));
    
    if (saveData[i].isUsed) {
        // Draw save info
        draw_text(250, _y, "Zone " + string(saveData[i].zone));
        DrawEmeralds(400, _y, saveData[i].emeralds);
    } else {
        draw_text(250, _y, "NO DATA");
    }
}
```

---

## 💡 Code Examples

### Loading Save Data

```gml
function LoadSaveSlot(slotNum) {
    var _save = {
        slotNumber: slotNum,
        isUsed: false,
        character: CHARACTER_SONIC,
        zone: 0,
        act: 1,
        emeralds: 0,
        lives: 3
    };
    
    var _filename = "save" + string(slotNum) + ".ini";
    
    if (file_exists(_filename)) {
        ini_open(_filename);
        _save.isUsed = ini_read_real("Data", "used", 0) == 1;
        _save.character = ini_read_real("Data", "character", CHARACTER_SONIC);
        _save.zone = ini_read_real("Data", "zone", 0);
        _save.act = ini_read_real("Data", "act", 1);
        _save.emeralds = ini_read_real("Data", "emeralds", 0);
        _save.lives = ini_read_real("Data", "lives", 3);
        ini_close();
    }
    
    return _save;
}
```

### Saving Game Data

```gml
function SaveGameToSlot(slotNum) {
    var _filename = "save" + string(slotNum) + ".ini";
    
    ini_open(_filename);
    ini_write_real("Data", "used", 1);
    ini_write_real("Data", "character", global.player_character);
    ini_write_real("Data", "zone", global.current_zone);
    ini_write_real("Data", "act", global.current_act);
    ini_write_real("Data", "emeralds", global.emeralds);
    ini_write_real("Data", "lives", global.lives);
    ini_close();
}
```

### Deleting a Save (Safe Method)

```gml
// Hold DELETE for 2 seconds
deleteHoldTime = 0;
deleteThreshold = 2 * room_speed;

// In Step Event
if (keyboard_check(vk_delete) && saveData[selectedSlot].isUsed) {
    deleteHoldTime++;
    if (deleteHoldTime >= deleteThreshold) {
        DeleteSaveSlot(selectedSlot);
        deleteHoldTime = 0;
    }
} else {
    deleteHoldTime = 0;
}
```

---

## 🎯 Advanced Features

### Save Validation

```gml
function ValidateSaveData(save) {
    if (save.lives < 0 || save.lives > 999) return false;
    if (save.zone < 0 || save.zone > MAX_ZONES) return false;
    if (save.act < 1 || save.act > 3) return false;
    return true;
}
```

### Corrupted Save Handling

```gml
function LoadSaveSlotSafe(slotNum) {
    var _save = LoadSaveSlot(slotNum);
    
    if (_save.isUsed && !ValidateSaveData(_save)) {
        show_debug_message("Save corrupted!");
        _save.isUsed = false;
        BackupCorruptedSave(slotNum);
    }
    
    return _save;
}
```

---

## 🎨 Visual Polish

Add these for better player experience:

1. **Animations**
   - Pulsing cursor
   - Fade transitions
   - Character icon animations

2. **Sound Effects**
   - Navigation beeps
   - Selection confirmation
   - Error sounds

3. **Visual Feedback**
   - Highlighted selected slot
   - Progress bars for emeralds
   - Character portraits

4. **Particles**
   - Sparkles on selection
   - Emerald shine effects

---

## ✅ Testing Checklist

Before release, verify:

- [ ] All slots selectable with keyboard/gamepad
- [ ] New game creates save properly
- [ ] Loading restores all data correctly
- [ ] Delete requires confirmation
- [ ] Corrupted saves handled gracefully
- [ ] Navigation wraps correctly
- [ ] Sound effects work
- [ ] Visual feedback is clear
- [ ] Can return to title
- [ ] Saves persist between sessions
- [ ] Multiple slots don't interfere
- [ ] UI scales at different resolutions

---

## 🎓 Best Practices

1. **Clear Feedback** - Make selection obvious
2. **Prevent Accidents** - Require confirmation for deletions
3. **Show Progress** - Display meaningful indicators
4. **Handle Errors** - Validate and recover from corruption
5. **Test Thoroughly** - Cover all edge cases
6. **Support Multiple Inputs** - Keyboard, gamepad, mouse
7. **Maintain Backups** - Keep emergency save copies
8. **Framework Integration** - Use Orbinaut's existing systems
9. **Follow Conventions** - Match framework naming/structure
10. **Performance** - Load efficiently, cache when possible

---

## 📚 Framework-Specific Tips

For Orbinaut Framework 2:

- Use existing `global` variables (lives, emeralds, etc.)
- Integrate with built-in character system
- Hook into zone completion events for auto-save
- Extend framework objects rather than creating from scratch
- Follow the framework's naming conventions
- Don't break existing framework features

---

## 🔗 Resources

- **Orbinaut Framework 2 Documentation**
- **Game Maker Manual** - File I/O functions
- **Reference Games:**
  - Sonic 3 & Knuckles (save select design)
  - Sonic Mania (modern implementation)
  - Sonic Advance series (multiple save slots)

---

## 💬 Discussion Questions

**For the community:**

1. What additional features would you add to a save select menu?
2. How do you handle save slot migration when updating your game?
3. What's your preferred visual style for save menus?
4. Any tips for optimizing save/load performance?
5. How do you handle cloud saves or cross-platform saves?

---

## 🤝 Contributing

If you've implemented a save select menu in your Orbinaut Framework 2 game:

- Share your approach and code snippets
- Post screenshots of your menu design
- Discuss challenges you encountered
- Help others troubleshoot their implementations

---

## 📝 Summary

A well-implemented save select menu enhances player experience by:
- Allowing multiple save files
- Providing clear progress visibility
- Preventing accidental data loss
- Maintaining data integrity
- Following Sonic game conventions

Start with the basic implementation above, test thoroughly, and gradually add polish and features!

---

**Questions? Drop them below! 👇**

Share your save select menu implementations, ask for help, or discuss improvements!
