# Vi Editor Quick Start Guide

A beginner's guide to navigating and using the vi text editor in Linux.

## Understanding Vi Modes

Vi has two main modes:
- **Normal Mode** (default): For navigation and commands
- **Insert Mode**: For typing/editing text

## Getting Started

### Opening a File
```bash
vi filename.txt
```

### The Two Most Important Keys
- **`i`** - Enter Insert Mode (so you can type)
- **`Esc`** - Return to Normal Mode (to save, exit, or navigate)

## Basic Workflow

1. Open file with `vi filename.txt`
2. Press `i` to start typing
3. Type your content
4. Press `Esc` when done editing
5. Type `:wq` and press `Enter` to save and exit

## Essential Commands

### Entering Insert Mode (from Normal Mode)
- `i` - Insert at cursor
- `a` - Insert after cursor
- `o` - Open new line below

### Saving and Exiting (from Normal Mode)
- `:w` - Save (write) file
- `:q` - Quit vi
- `:wq` - Save and quit
- `:q!` - Quit without saving (force quit)
- `ZZ` - Save and quit (shortcut)

### Navigation (in Normal Mode)
- `h` - Move left
- `j` - Move down
- `k` - Move up
- `l` - Move right
- Arrow keys also work!

### Editing (in Normal Mode)
- `x` - Delete character under cursor
- `dd` - Delete entire line
- `u` - Undo last change
- `Ctrl + r` - Redo

## Quick Reference Card

| Action | Command |
|--------|---------|
| Start editing | `i` |
| Stop editing | `Esc` |
| Save | `:w` |
| Quit | `:q` |
| Save & Quit | `:wq` |
| Quit without saving | `:q!` |
| Undo | `u` |
| Delete line | `dd` |

## Common Beginner Mistakes

❌ **Problem**: Can't type anything  
✅ **Solution**: Press `i` to enter Insert Mode

❌ **Problem**: Keys doing weird things instead of typing  
✅ **Solution**: You're in Normal Mode, press `i`

❌ **Problem**: Can't save or exit  
✅ **Solution**: Press `Esc` first, then type `:wq`

## Practice Exercise

Try this to get comfortable:
```bash
# 1. Create a practice file
vi practice.txt

# 2. Press 'i' to enter Insert Mode
# 3. Type: "Hello, I'm learning vi!"
# 4. Press 'Esc' to exit Insert Mode
# 5. Type ':wq' and press Enter to save and exit
```

## Pro Tip

If you get stuck or confused, press `Esc` a few times to make sure you're in Normal Mode, then type `:q!` to quit without saving.

---

**Remember**: `Esc` is your friend. When in doubt, press `Esc` to return to Normal Mode!
