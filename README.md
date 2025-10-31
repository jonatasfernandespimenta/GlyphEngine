# GlyphEngine 🔥

> A powerful, reusable Python engine for creating top-down games in the terminal

GlyphEngine is a modular game engine designed specifically for building ASCII/terminal-based games with a focus on top-down perspective gameplay. It provides core systems for game loop management, map handling, player mechanics, UI interactions, procedural generation, and advanced UI components including selectable menus and grid editors.

---

## 🎮 Features

- **Complete Game Loop Management** - Built-in game client with update/draw cycles
- **Flexible Map System** - Base classes for creating custom maps with collision detection
- **Procedural Generation** - Backtracking algorithm for generating maze-like maps
- **Player Management** - Full player class with movement, inventory, stats, and leveling
- **UI Components** - Reusable UI systems for character creation, NPC interactions, selectable menus, and grid editors
- **State Management** - Persistent system registration for farms, quests, etc.
- **Map Transitions** - Portal system for seamless map changes
- **Terminal Rendering** - Built on the `blessed` library for cross-platform terminal graphics

---

## 📦 Installation

### Requirements

```bash
pip install blessed
```

### Project Structure

```
engine/
├── core/
│   ├── game_client.py       # Main game loop and state management
│   ├── state_manager.py     # Persistent system registry
│   └── player.py            # Base player class with movement, inventory, stats
├── maps/
│   ├── map.py               # Base Map class
│   ├── procedural_board.py  # Procedural map generation
│   └── map_transition.py    # Portal/transition system
├── ui/
│   ├── interaction_ui.py    # Generic NPC/shop interaction UI
│   ├── character_creation_ui.py  # Character creation flow
│   ├── selectable_menu.py   # Menu system with keyboard navigation
│   └── grid_editor.py       # Grid-based element editor with draggable components
└── network/
    └── (network abstractions)
```

---

## 🚀 Quick Start

### Basic Game Setup

```python path=null start=null
from engine.core.game_client import GameClient
from engine.core.player import Player
from engine.maps.map import Map

# Create game client
client = GameClient()

# Create a simple map
class MyMap(Map):
    def __init__(self):
        super().__init__(width=50, height=20)
        self.createBoard()
    
    def createBoard(self):
        self.lines = [['.' for _ in range(self.windowWidth)] 
                      for _ in range(self.windowHeight)]
        # Add walls around the border
        for i in range(self.windowHeight):
            self.lines[i][0] = '#'
            self.lines[i][-1] = '#'
        for j in range(self.windowWidth):
            self.lines[0][j] = '#'
            self.lines[-1][j] = '#'
    
    def printBoard(self, term):
        for line in self.lines:
            print(''.join(line))
    
    def init(self, players, term):
        self.printBoard(term)

# Create player
my_map = MyMap()
player = Player(
    lines=my_map.getLines(),
    windowWidth=my_map.getWindowWidth(),
    windowHeight=my_map.getWindowHeight(),
    playerPosition=[10, 10],
    name="Hero",
    term=client.term
)

# Setup and run
client.setCurrentMap(my_map)
client.setPlayer(player)
client.gameLoop()
```

---

## 📖 Core Components

### GameClient

The main game controller that manages the game loop, systems, and state.

#### API

```python path=null start=null
client = GameClient()

# System registration
client.registerSystem('farm', farm_system)
client.registerSystem('quest', quest_system)

# Access systems
farm = client.getSystem('farm')

# Setup
client.setCurrentMap(city_map)
client.setPlayer(player)

# Run game
client.gameLoop()

# Stop game
client.stop()
```

#### Methods

| Method | Description |
|--------|-------------|
| `registerSystem(name, system)` | Register a game system (farm, quest, etc.) |
| `getSystem(name)` | Retrieve a registered system |
| `setCurrentMap(map_obj)` | Set the active map |
| `setPlayer(player)` | Set the main player |
| `draw()` | Render the current frame |
| `update()` | Update game state (called every frame) |
| `gameLoop()` | Start the main game loop |
| `stop()` | Stop the game loop |
| `showGameOver()` | Display game over screen |

---

### StateManager

Manages persistent game systems that need to maintain state across different maps and sessions.

```python path=null start=null
state_manager = StateManager()

# Register systems
state_manager.registerSystem('farm', Farm())
state_manager.registerSystem('quest', QuestManager())

# Access systems
farm = state_manager.getSystem('farm')

# Update all systems (called automatically by GameClient)
state_manager.update()
```

---

### Player

Base player class with movement, inventory, stats, and progression systems.

#### Initialization

```python path=null start=null
player = Player(
    lines=map.getLines(),           # 2D board array
    windowWidth=50,                 # Map width
    windowHeight=20,                # Map height
    playerPosition=[10, 10],        # Starting [y, x]
    name="Hero",                    # Player name
    term=terminal,                  # Blessed Terminal instance
    blockers=['#', 'W']            # Characters that block movement
)
```

#### Stats & Progression

```python path=null start=null
# Stats
player.getHp()                  # Current HP
player.getMaxHp()               # Max HP
player.getAttack()              # Attack power
player.getDefense()             # Defense power
player.setHp(80)                # Set HP (capped at maxHp)

# Leveling
player.getLevel()               # Current level
player.getXp()                  # Current XP
player.addXp(50)                # Add XP (auto-levels up)
player.levelUp()                # Manual level up
```

#### Movement

```python path=null start=null
# Movement handled automatically with WASD or arrow keys
player.movePlayer()

# With network callback for multiplayer
player.movePlayer(network_callback=lambda pos: send_to_server(pos))

# Position management
pos = player.getPlayerPosition()  # Returns [y, x]
player.setPlayerPosition([5, 10])

# Board updates (when changing maps)
player.setBoard(new_lines, new_width, new_height)

# Drawing
player.drawPlayer()    # Place player on board
player.removePlayer()  # Remove player from current position
```

#### Inventory System

```python path=null start=null
# Add items
item = {'name': 'Health Potion', 'hp': 50}
player.addToInventory(item)

# Get inventory (with quantities)
inventory = player.getInventory()

# Equip/use items
player.equipItem(0)  # Apply item effects

# Drop items
player.dropItem(0)

# Toggle inventory UI
player.getIsInventoryOpen()
player.setIsInventoryOpen(True)
```

#### Gold & Notifications

```python path=null start=null
# Gold management
player.getGold()
player.addGold(100)

# Notifications
player.showNotification("Quest completed!", duration=3.0)
message = player.getNotification()  # Returns '' if expired
```

---

### Map

Base class for all game maps. Subclasses must implement core methods.

#### Creating a Custom Map

```python path=null start=null
from engine.maps.map import Map

class CityMap(Map):
    def __init__(self):
        super().__init__(width=80, height=30)
        self.createBoard()
    
    def createBoard(self):
        """Initialize the map layout"""
        self.lines = [['.' for _ in range(self.windowWidth)] 
                      for _ in range(self.windowHeight)]
        # Add walls, buildings, NPCs, etc.
        # Use self.placeArt() for ASCII art
    
    def printBoard(self, term):
        """Render the map"""
        for line in self.lines:
            colored_line = ''.join(
                term.green(c) if c == '.' else 
                term.white(c) for c in line
            )
            print(colored_line)
    
    def init(self, players, term):
        """Draw the map and players"""
        self.printBoard(term)
        for player in players:
            player.drawPlayer()
    
    def handleCollisions(self, player, draw, term):
        """Handle collisions with NPCs, enemies, chests"""
        pos = player.getPlayerPosition()
        char = self.lines[pos[0]][pos[1]]
        
        if char == 'N':  # NPC
            self.startNPCDialogue(player, term)
        elif char == 'E':  # Enemy
            self.startCombat(player, term)
    
    def checkPortalTransition(self, player):
        """Return MapTransition if player is on a portal"""
        pos = player.getPlayerPosition()
        if self.lines[pos[0]][pos[1]] == 'P':
            return DungeonTransition(dungeon_map, spawn_pos=[5, 5])
        return None
```

#### Map API

| Method | Description |
|--------|-------------|
| `createBoard()` | Initialize the board layout (must implement) |
| `printBoard(term)` | Render the map to terminal (must implement) |
| `init(players, term)` | Draw map and players (must implement) |
| `handleCollisions(player, draw, term)` | Handle NPC/enemy/object interactions |
| `checkPortalTransition(player)` | Check for map transitions |
| `placeArt(startY, startX, art)` | Place ASCII art on the board |
| `convertArtToBoardItem(art)` | Convert art string to 2D array |

---

### ProceduralBoard

Generates maze-like maps using a backtracking algorithm.

```python path=null start=null
from engine.maps.procedural_board import ProceduralBoard

# Create empty board
board = [['.' for _ in range(width)] for _ in range(height)]

# Generate
proc_gen = ProceduralBoard(board, width, height)
proc_gen.procedurelyGeneratedBoard()

# Get result
generated_board = proc_gen.getBoard()
```

The algorithm:
1. Starts at a random cell
2. Carves paths by moving to unvisited neighbors
3. Backtracks when stuck
4. Converts 'X' to walls '#' and 'O' to floor '.'

---

### MapTransition

System for transitioning between maps via portals.

```python path=null start=null
from engine.maps.map_transition import MapTransition

class DungeonEntrance(MapTransition):
    def __init__(self, dungeon_map):
        super().__init__(
            destination_map=dungeon_map,
            player_spawn_position=[5, 5]
        )
    
    def execute(self, player, term):
        """Show transition message and update player board"""
        print(term.center(term.yellow("Entering the dungeon...")).rstrip())
        term.inkey(timeout=1.5)
        
        # Update player's board reference
        player.setBoard(
            self.destination_map.getLines(),
            self.destination_map.getWindowWidth(),
            self.destination_map.getWindowHeight()
        )
        
        # Spawn player at new position
        player.setPlayerPosition(self.player_spawn_position)

# In your Map class
def checkPortalTransition(self, player):
    pos = player.getPlayerPosition()
    if self.lines[pos[0]][pos[1]] == 'D':  # Dungeon entrance
        return DungeonEntrance(self.dungeon_map)
    return None
```

---

### InteractionUI

Generic UI system for NPC dialogues, shops, and interactive objects.

```python path=null start=null
from engine.ui.interaction_ui import InteractionUI

# Shop example
shop_config = {
    'art': """
    ╔════════════╗
    ║   SHOP     ║
    ╚════════════╝
    """,
    'message': 'Welcome to the shop!',
    'show_gold': True,
    'items': [
        {
            'name': 'Health Potion',
            'description': 'Restores 50 HP',
            'price': 25,
            'owned': False,
            'action': lambda item, ui: ui.player.addToInventory(item)
        },
        {
            'name': 'Iron Sword',
            'description': '+5 Attack',
            'price': 100,
            'owned': False,
            'action': lambda item, ui: ui.player.setAttack(
                ui.player.getAttack() + 5
            )
        }
    ],
    'options': [
        {
            'key': 'h',
            'label': 'Help',
            'action': lambda: print('Press number keys to buy items!')
        }
    ]
}

shop_ui = InteractionUI(player, term, shop_config)
shop_ui.open()  # Blocks until user exits with 'Q'
```

#### Configuration

| Field | Type | Description |
|-------|------|-------------|
| `art` | str | ASCII art displayed at top |
| `message` | str | Message/dialogue text |
| `show_gold` | bool | Display player's gold |
| `items` | list[dict] | Shop items (see below) |
| `options` | list[dict] | Action buttons (see below) |

#### Item Format

```python path=null start=null
{
    'name': 'Item Name',
    'description': 'Item description (optional)',
    'price': 100,
    'owned': False,  # Tracked automatically
    'action': lambda item, ui: do_something()
}
```

#### Option Format

```python path=null start=null
{
    'key': 'h',  # Keyboard shortcut
    'label': 'Help',
    'action': lambda: do_something()
}
```

---

### CharacterCreationUI

Reusable UI for character creation with name input and class selection.

```python path=null start=null
from engine.ui.character_creation_ui import CharacterCreationUI

char_ui = CharacterCreationUI(term)

# Complete flow
character = char_ui.create(
    game_title='=== MY RPG ===',
    name_prompt='Enter your name: ',
    class_title='=== CHOOSE YOUR CLASS ===',
    classes=[
        {
            'key': '1',
            'name': 'Warrior',
            'description': 'Strong and durable',
            'stats': 'HP: 120 | Attack: 12 | Defense: 8',
            'color': 'bold_red',
            'id': 'warrior'
        },
        {
            'key': '2',
            'name': 'Rogue',
            'description': 'Fast and agile',
            'stats': 'HP: 80 | Attack: 15 | Defense: 4',
            'color': 'bold_green',
            'id': 'rogue'
        },
        {
            'key': '3',
            'name': 'Mage',
            'description': 'Master of magic',
            'stats': 'HP: 70 | Attack: 18 | Defense: 3',
            'color': 'bold_blue',
            'id': 'mage'
        }
    ],
    max_name_length=20
)

# Returns: {'name': 'Hero', 'class': 'warrior'}
player_name = character['name']
player_class = character['class']

# Or use individual methods
name = char_ui.getName(title='=== MY RPG ===')
selected_class = char_ui.selectClass(
    title='=== CHOOSE YOUR CLASS ===',
    classes=classes
)
```

---

## 🎛️ SelectableMenu

A flexible menu system with keyboard navigation, customizable styling, and callback support.

```python path=null start=null
from engine.ui.selectable_menu import SelectableMenu

# Create menu
menu = SelectableMenu(
    term=term,
    title='Main Menu',
    show_numbers=True  # Show 1, 2, 3... prefixes
)

# Add menu items with callbacks
menu.add_item('New Game', callback=lambda item: start_new_game())
menu.add_item('Load Game', callback=lambda item: load_game())
menu.add_item('Settings', callback=lambda item: open_settings())
menu.add_item('Exit', callback=lambda item: exit_game())

# Customize appearance
menu.marker_selected = ' ▶ '
menu.marker_unselected = '   '
menu.color_selected = 'black_on_yellow'
menu.color_unselected = 'white'

# Render at position
menu.render(x=10, y=5, width=30)

# Handle input in game loop
key = term.inkey()
if menu.handle_input(key):
    # An action was triggered
    pass
```

### MenuItem API

Each menu item supports additional properties:

```python path=null start=null
item = menu.add_item('Special Item', data={'value': 100})
item.is_selectable = False  # Make it non-selectable (visual only)
item.is_visible = False     # Hide item from menu
```

### Navigation Methods

| Method | Description |
|--------|-------------|
| `add_item(name, callback, data)` | Add a menu item with optional callback and data |
| `clear_items()` | Remove all menu items |
| `get_selected_item()` | Get currently selected MenuItem |
| `next_item()` | Move selection down (skips non-selectable) |
| `previous_item()` | Move selection up (skips non-selectable) |
| `select_by_number(index)` | Select item by index (0-based) |
| `execute_selected()` | Run callback of selected item |
| `render(x, y, width)` | Draw menu at position |
| `handle_input(key)` | Process keyboard input (W/S/arrows/numbers/Enter) |

### Input Handling

- **Arrow Keys / W/S**: Navigate up/down
- **Enter / Space**: Select current item
- **Number Keys**: Direct selection (if `show_numbers=True`)

---

## 🎨 GridEditor

A grid-based editor for positioning and managing draggable elements, perfect for level editors or UI layout tools.

```python path=null start=null
from engine.ui.grid_editor import GridEditor
from engine.ui.draggable import DraggableElement

# Create grid editor
editor = GridEditor(term=term, width=80, height=30)

# Create draggable elements
player_element = DraggableElement(
    element_id='player',
    x=10, y=10,
    art="""🧙"""
)

enemy_element = DraggableElement(
    element_id='enemy',
    x=20, y=15,
    art="""👾"""
)

# Add elements to editor
editor.add_element(player_element)
editor.add_element(enemy_element)

# Render grid with elements
editor.render_grid()

# Handle movement in game loop
key = term.inkey()
if editor.handle_movement(key):
    # Element was moved, re-render
    editor.render_grid()

# Cycle through elements
if key == '\t':  # Tab key
    editor.next_element()
elif key.name == 'KEY_BTAB':  # Shift+Tab
    editor.previous_element()
```

### GridEditor API

| Method | Description |
|--------|-------------|
| `add_element(element)` | Add a DraggableElement to the grid |
| `remove_element(element_id)` | Remove element by ID |
| `get_selected_element()` | Get currently selected element |
| `next_element()` | Select next element |
| `previous_element()` | Select previous element |
| `create_grid(floor_char, wall_h, wall_v)` | Generate grid background |
| `place_element_on_grid(element)` | Place element art on grid |
| `render_elements()` | Render all elements on grid |
| `render_grid()` | Full render (grid + elements) |
| `handle_movement(key)` | Move selected element (WASD/arrows) |

### DraggableElement

```python path=null start=null
class DraggableElement:
    def __init__(self, element_id: str, x: int, y: int, art: str):
        self.element_id = element_id  # Unique identifier
        self.x = x                    # Grid X position
        self.y = y                    # Grid Y position
        self.art = art                # ASCII art string
        self.bounds = None            # Optional (min_x, min_y, max_x, max_y)
    
    def move(self, dx: int, dy: int) -> bool:
        """Move element, respecting bounds. Returns True if moved."""
        pass
    
    def get_art_lines(self) -> List[List[str]]:
        """Get art as 2D character array"""
        pass
```

### Custom Grid Styling

```python path=null start=null
# Customize grid appearance
editor.create_grid(
    floor_char='.',      # Background character
    wall_char_h='═',    # Horizontal walls
    wall_char_v='║'     # Vertical walls
)
# Creates grid with ╔╗╚╝ corners automatically
```

---

## 🎯 Advanced Examples

### Example 1: Complete Game with Multiple Systems

```python path=null start=null
from engine.core.game_client import GameClient
from engine.core.player import Player
from engine.maps.map import Map
from engine.ui.character_creation_ui import CharacterCreationUI

# Farm system
class FarmSystem:
    def __init__(self):
        self.plots = {}
    
    def purchasePlot(self, position):
        self.plots[tuple(position)] = None
    
    def plantCrop(self, position, crop_type):
        self.plots[tuple(position)] = {
            'type': crop_type,
            'planted_at': time.time()
        }
    
    def update(self):
        # Update crop growth
        pass

# Create game
client = GameClient()

# Character creation
char_ui = CharacterCreationUI(client.term)
character = char_ui.create(
    game_title='=== MY FARM RPG ===',
    classes=[...]
)

# Create player
player = Player(
    lines=city.getLines(),
    windowWidth=city.getWindowWidth(),
    windowHeight=city.getWindowHeight(),
    playerPosition=[15, 15],
    name=character['name'],
    term=client.term
)

# Register systems
farm = FarmSystem()
client.registerSystem('farm', farm)

# Setup and run
client.setCurrentMap(city)
client.setPlayer(player)
client.gameLoop()
```

### Example 2: Map with Portals

```python path=null start=null
from engine.maps.map import Map
from engine.maps.map_transition import MapTransition

class CityToDungeonTransition(MapTransition):
    def execute(self, player, term):
        print(term.center(term.red("Entering dark dungeon...")).rstrip())
        term.inkey(timeout=1.5)
        
        player.setBoard(
            self.destination_map.getLines(),
            self.destination_map.getWindowWidth(),
            self.destination_map.getWindowHeight()
        )
        player.setPlayerPosition(self.player_spawn_position)

class CityMap(Map):
    def __init__(self, dungeon_map):
        super().__init__(80, 30)
        self.dungeon_map = dungeon_map
        self.createBoard()
    
    def createBoard(self):
        # ... create city layout
        self.lines[25][40] = 'D'  # Dungeon entrance
    
    def checkPortalTransition(self, player):
        pos = player.getPlayerPosition()
        if self.lines[pos[0]][pos[1]] == 'D':
            return CityToDungeonTransition(
                self.dungeon_map,
                player_spawn_position=[5, 5]
            )
        return None
```

### Example 3: Interactive NPC with InteractionUI

```python path=null start=null
from engine.ui.interaction_ui import InteractionUI

def create_blacksmith_ui(player, term):
    def upgrade_weapon():
        if player.getGold() >= 50:
            player.addGold(-50)
            player.setAttack(player.getAttack() + 5)
            print(term.green("Weapon upgraded!"))
        else:
            print(term.red("Not enough gold!"))
    
    config = {
        'art': """
        ⚒️  BLACKSMITH  ⚒️
        """,
        'message': "I can forge mighty weapons for the right price!",
        'show_gold': True,
        'options': [
            {
                'key': 'u',
                'label': 'Upgrade Weapon (50 gold)',
                'action': upgrade_weapon
            }
        ]
    }
    
    ui = InteractionUI(player, term, config)
    return ui

# In Map.handleCollisions()
def handleCollisions(self, player, draw, term):
    pos = player.getPlayerPosition()
    if self.lines[pos[0]][pos[1]] == 'B':  # Blacksmith
        ui = create_blacksmith_ui(player, term)
        ui.open()
```

---

## 🎨 Art Utilities

Place ASCII art easily on your maps:

```python path=null start=null
# In your Map class
house_art = """
    🏠
   /  \\
  /____\\
  | 🚪 |
  |____|
"""

art_2d = self.convertArtToBoardItem(house_art)
self.placeArt(startY=10, startX=20, art=art_2d)
```

---

## 🔧 Customization

### Custom Player Classes

```python path=null start=null
from engine.core.player import Player

class WarriorPlayer(Player):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.maxHp = 150
        self.hp = 150
        self.attack = 15
        self.defense = 10
    
    def levelUp(self):
        """Custom leveling for warrior"""
        super().levelUp()
        self.maxHp += 15  # Extra HP gain
        self.hp = self.maxHp
```

### Custom Game Systems

```python path=null start=null
class QuestSystem:
    def __init__(self):
        self.quests = []
        self.completed = []
    
    def addQuest(self, quest):
        self.quests.append(quest)
    
    def completeQuest(self, quest_id):
        quest = next(q for q in self.quests if q['id'] == quest_id)
        self.quests.remove(quest)
        self.completed.append(quest)
    
    def update(self):
        # Check quest objectives each frame
        pass

# Register with client
client.registerSystem('quest', QuestSystem())
```

---

## 🎓 Best Practices

1. **Separation of Concerns**: Keep game logic separate from engine code
2. **Use StateManager**: Register any system that needs to persist across maps
3. **Map Transitions**: Always update player board reference when changing maps
4. **Collision Handling**: Implement `handleCollisions()` in Map subclasses
5. **Character Art**: Use a single character for the player (`self.playerChar = 'X'`)
6. **Blockers**: Define which characters block movement (`blockers=['#', 'W']`)
7. **Network Callbacks**: Pass callbacks to `movePlayer()` for multiplayer sync

---

## 🐛 Debugging Tips

```python path=null start=null
# Enable debug mode by printing to a log file
import sys
sys.stdout = open('debug.log', 'w')

# Check player position
print(f"Player at: {player.getPlayerPosition()}")

# Verify map size
print(f"Map: {map.getWindowWidth()}x{map.getWindowHeight()}")

# List registered systems
print(f"Systems: {list(client.state_manager.systems.keys())}")
```

---

## 📝 License

This engine is part of the CmdMMO project. Feel free to use and modify for your own terminal-based games!

---

## 🤝 Contributing

Contributions are welcome! Areas for improvement:
- Network multiplayer support
- Audio system (beeps/sounds)
- Save/load system
- Combat system base class
- Dialogue tree system
- Particle effects (animated ASCII)

---

## 🎮 Example Games

Check out the `game/` directory for a complete MMO implementation using GlyphEngine:
- Farm system with crop growth
- Procedural dungeons
- Combat system
- NPC interactions
- Inventory management
- Multiple interconnected maps

---

**Built with ❤️ for terminal game developers**

*Glyph - Forge your adventure in the terminal*
