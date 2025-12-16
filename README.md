# SHROOM RAIDER 

## USER MANUAL
This section explains how to run the base game of Shroom Raider and what the controls are.

### How To Run The Base Game
The game is primarily run from the command line in the directory `shroom_raider_files`.

1.  **Start Interactive Mode (Main Menu):**
    To start the game and load the main menu, run the script with no arguments:
    ```bash
    python3 shroom_raider.py
    ```
    or 

    ```bash
    python3 -m shrooom_raider
    ```

From the main menu, you can select a level by moving your character ('🧑') onto the corresponding numbered tile.
The main menu will be discussed in the bonus section.

2.  **Start Interactive Mode (Specific Level):**
    To directly load a specific level (e.g., `level1.txt`), run:
    ```bash
    python3 shroom_raider.py <stage_file>
    ```
    or

    ```bash
    python3 -m shroom_raider -f <stage_file>
    ```

3.  **Run Non-Interactive:**
    You can run the game non-interactively by providing a string of moves and specifying an output file. This mode is useful for testing the game.
    ```bash
    python3 shroom_raider.py -f <stage_file> -m <string_of_moves> -o <output_file>
    ```
    or

    ```bash
    python3 -m shroom_raider -f <stage_file> -m <string_of_moves> -o <output_file>
    ```

    * `-f`: Specifies the level file (e.g., `level1.txt`).
    * `-m`: Specifies the sequence of moves (e.g., `"wswdawdp"`).
    * `-o`: Specifies the name of the output file (e.g., `output.txt`), which will contain the final map state and win/loss status.

### Controls
The game uses single-character input for all actions:

| Key | Action | Description |
| :---: | :--- | :--- |
| **W** | Move Up | Move the character one tile up. |
| **A** | Move Left | Move the character one tile left. |
| **S** | Move Down | Move the character one tile down. |
| **D** | Move Right | Move the character one tile right. |
| **P** | Pick Up | Pick up an item (Axe '🪓' or Flamethrower '🔥') if you are standing on it and your inventory is empty. |
| **!** | Reset | Restarts the current level from the beginning. |
| **@** | Exit/Menu | Exits the game if you are in a level launched directly, or returns to the **Main Menu** if launched from the menu. |

Doing multi-character input for an action will still run the game normally until it hits an invalid input. 

Once the multi-character input executes its last input or until an invalid input is identified, it will print out the latest state of the game.

The controls are case-insensitive.

### Objects and Tiles
| Emoji | ASCII | Description |
| :---: | :--- | :--- |
| "🧑" | L | This is the character you are controlling |
#### Items
| Emoji | ASCII | Description |
| :---: | :--- | :--- |
| "🪓" | x | Cuts down the tree you move into |
| "🔥" | * | Burns down the tree you move into, including adjacent ones|
| "🍄" | + | Collectibles that serves as a win condition once all collected|

#### Obstacles and Spaces
| Emoji | ASCII | Description |
| :---: | :--- | :--- |
| "　" | . | A free space tile you can walk on.|
| "🌲" | T | An immovable obstacles that blocks movement unless you have an Axe or Flamethrower.|
| "🪨" | R | A movable obstacle as long the a free space tile or a water tile is in front. Otherwise immovable.|
| "🟦" | ~ | If the player moves into this tile, they lose.|
| "⬜" | _ | If a rock gets pushed to this tile, a paved tile is created. Acts the same way as a free space tile.|
### Main Menu


<img width="342" height="173" alt="image" src="https://github.com/user-attachments/assets/104466b5-11dd-4ae9-94e3-bef9a57d8682" />


In here, the user can access the 8 main levels of our game as well as the leaderboard, which can be accessed by going to the 9 tile. Each of the 8 levels have been designed to be very challenging and fun!

### Leaderboard


<img width="648" height="615" alt="image" src="https://github.com/user-attachments/assets/17afa55d-a26e-4c7c-9f28-75194d3eef76" />


### Levels

#### Level 1


<img width="381" height="227" alt="image" src="https://github.com/user-attachments/assets/9e4b97f5-d99f-4c71-ae79-b02733e970f4" />


#### Level 2


<img width="200" height="312" alt="image" src="https://github.com/user-attachments/assets/b67ec20a-3e5c-4ee6-a18a-f19143881692" />




## Code 

### Gameplay `gameplay.py`
The file is responsible for running the entire game together. It has one class function `Game`, which holds ten functions tying everything together.

#### Initializing Game (`initialize_game`)

Parses command-line arguments using `argparse`. This function has two modes that the player can initiate:

1. Interactive Mode: 
    * If no arguments are provided, it proceeds to the main menu.
    * If a level was provided, it immediate proceeds to the exact level, ignoring the the process to run and print the main menu.
    
2. Non-Interactive: If `-f` (file) and `-m` (moves) are provided, it sets `self.is_interactive = False` and proceeds directly to `play_level`, outputting the result to a file specified by `-o`.

If in interactive mode, it calls `start_menu()` to load the main menu.

#### Opening Menu (`start_menu`)

This function initializes runs before the `main_menu` function to set the following necessary conditions for the `main_menu` file:

* Loads the `main_menu.txt` level file into `self.level_as_list`.
* Initializes the map coordinates for menu elements (stages, leaderboard).
* Plays a start game animation and prints the initial menu screen.
* Calls `main_menu("")` to enter the menu interaction loop.

#### Main Menu Loop (`main_menu`)

This function continuously handles player input while in the menu:

* Laro Movement: Processes player input using `self.player.movement`.
* Collision Checks:
    * Rocks (`R`): Calls `self.rockmechanics.rock_movement` to check if the rock can be pushed.
    * Trees (`T`): Calls `self.treemechanics.item_tree_iteraction` to check if an axe can be used.
* Tile Update: If movement is valid (`can_move` is True), Laro's coordinates are updated in the map.
* Menu Options:
    * Stage Selection: Calls `_check_stage_selection`. If Laro moves onto a stage number (1-8), it returns the level file name, which exits the menu loop and triggers the level load.
    * Leaderboard (9): Calls `display_leaderboard()`, then calls `_reload_menu()` to refresh the menu state.
    * Exit (@): Exits the game.
* The function recursively calls itself (`return self.main_menu(remaining_actions)`) until a stage is selected or the game is exited.

#### Reloading Menu (`_reload_menu`)

* This is called to refresh the menu display, primarily after the Leaderboard (9) is displayed.
* It re-reads the `main_menu.txt` to reset the map and re-initializes all coordinate objects (`self.coords`, `self.player.coords`, etc.) to ensure a clean state for the menu coordinates.

#### Stage Selection (`_check_stage_selection`)

* Checks if Laro's new coordinates (`new_laro_coords`) overlap with the coordinate sets for menu options 1 through 8.
* If a stage is selected, it clears the current level data and coordinates and returns the corresponding level file path, initiating level loading.

#### Initializing Level (`play_level`)

* Loads the selected stage file into `self.level_file.read_text()`.
* Resets the move counter, mushroom eaten count, and plays a "start level" animation.
* Mapping: Calls `self.coords.map_coords_generator` to locate all game objects (Rocks, Trees, Mushrooms, Laro) on the new map.
* Mode Handling:
    * Non-Interactive: Calls `self.gameplay` with the entire string of moves (`self.args.moves`). After gameplay, it determines "CLEAR" or "NO CLEAR" status, writes the result and the final map state to the output file.
    * Interactive: Prints the initial level and calls `self.gameplay("")` to start the game loop.

#### Gameplay Loop (`gameplay`)

This function is responsible for tying all the functions together and looping the game after the player inputs an action.

* Move Counter: Increments `self.move_counter` for each action.
* Laro Movement: Gets input or uses remaining actions, processes movement.
* Stage Transition: Checks for exit (`@`) or restart (`!`) actions, calling `_stage_transition` if found.
* Collision/Interaction (`_determine_can_move`):
    * Rock (`R`): Tries to push the rock. Movement is only valid if the push succeeds.
    * Tree (`T`): Tries to cut the tree with an axe (if available). Movement is valid if the tree is removed or if no interaction is needed.
    * Empty Space/Other: Movement is valid.
* Tile Update: If movement is valid, Laro's location is updated, potentially removing a mushroom or replacing a rock's previous location.
* Item Pickup: Calls `self.inventory.pickup_item` to collect items (e.g., axes).
* After Game Check (`_maybe_handle_after_game`): Checks if the level is complete (all mushrooms eaten). If complete, it triggers the leaderboard input.
* Printing: If interactive, it clears the screen, prints the updated level, and displays the current score.
* The function recursively calls itself until the level is completed or exited.

#### Stage Transitions (`_stage_transition`)

Handles exiting or restarting a stage:

* Clears current level coordinates and animations.
* If action is Exit (`@`):
    * If running interactively from the main menu (no arguments), it returns to `initialize_game` to show the main menu again.
    * If running with arguments (or from a top-level start), it exits the system.
* If action is Restart (`!`):
    * Plays a "restart level" animation and calls `self.play_level()` to reload the current stage from its file.

#### Movement Determination (`_determine_can_move`)

Helper method that checks whether Laro can move to a target coordinate:

* Checks if the target contains a Rock (`R`) and attempts to push it using `rock_movement`.
* Checks if the target contains a Tree (`T`) and attempts interaction using `item_tree_iteraction`.
* Returns `True` if movement is valid, `False` otherwise.

#### After Game and Leaderboard Input (`_maybe_handle_after_game`)

* `self.after_game.after_game_condition()` checks if Laro has fallen into water (`~`) or collected all mushrooms.
* If all mushrooms have been eaten:
    * The final score (`10000 - self.move_counter * 10`) is calculated and printed.
    * The player is prompted to `Enter your name`.
    * `add_score()` is called to save the score to the leaderboard for that level.
* Returns `1` if game over condition is met, `0` otherwise.

### Main Menu `main_menu.py`
This file is responsible for handling the functions for the `main_menu` function in the class Game. This is feature require a separate file as it handles features that the base game does cannot process (e.g., `"1️⃣`).

This file contains one class with two assignments and four nested functions:

#### Class Variables

* **Menu Coordinates (`menu_coords`)**: A `ClassVar` dictionary that maps the strings '1' to '9' to their respective coordinate sets. Each number tile on the main menu is tracked here.

* **Level Loader (`level_loader`)**: A `ClassVar` dictionary which holds the mapping for levels 1-8. Maps menu numbers to their corresponding level file names (e.g., '1' → 'level1.txt').

#### Constructor (`__init__`)

Initializes the `MainMenu` class with a reference to the shared `Coordinates` object, allowing the menu to access and modify game coordinates.

#### Clear Menu Coordinates (`_clear_menu_coords`)

Private helper method that clears all coordinate sets within `self.menu_coords` (numbers 1-9) to provide a fresh state. Also resets Laro's position to `(0, 0)` if present in coordinates.

#### Process Character (`_process_char`)

Private helper method that processes a single character at a given coordinate:

* If the character is a menu number ('1'-'9'), adds the coordinate to the corresponding set in `menu_coords`.
* If the character is Laro ('L'), sets Laro's position in `self.coords.coords["L"]` and adds the coordinate to the dot coordinates set.
* For other characters that exist in the coordinate dictionary, adds them to their respective coordinate sets.

#### Menu Coords Generator (`menu_coords_generator`)

The primary function for parsing the `main_menu.txt` map:

1. Calls `_clear_menu_coords()` to reset menu coordinate tracking.
2. Clears all existing coordinates in `self.coords.coords` (sets and lists are cleared, Laro position reset to `(0, 0)`).
3. Iterates through the entire menu map grid.
4. For each tile, calls `_process_char()` to map the character to its coordinate set.

This ensures every element in the menu (stage numbers, trees, Laro) is properly tracked in the coordinate system.

#### Menu Printer (`menu_printer`)

Takes the current `level_as_list` and renders it with emoji representations:

* Creates a mapping dictionary from ASCII characters to emojis (e.g., '1' → '1️⃣', 'T' → '🌲', 'L' → '🧑').
* Iterates through each line in the level.
* Converts each character to its emoji equivalent using the mapping.
* Prints the emoji-rendered line to display the visual menu.

This provides the visual representation of the main menu that players see in the terminal.

### Coordinates `coordinates.py`

This file contains the `Coordinates` dataclass that manages all coordinate tracking for game entities. It stores positions of all objects (trees, rocks, mushrooms, etc.) and provides methods to update and organize them.

#### Dataclass Structure

The `Coordinates` class uses a `coords` dictionary that maps string keys to coordinate values:

* Single entity: `'L'` maps to a tuple `(row, col)` representing Laro's position.
* Multiple entities: `'T'`, `'R'`, `'~'`, etc. map to sets of tuples representing all positions of that entity type.
* Tree groups: `'T'` can also map to a list of sets, where each set represents a connected group of trees.

#### Reset Coordinates (`_reset_coords`)

Private helper method that clears all coordinate buckets:

* For sets and lists: calls `.clear()` to empty them.
* For Laro's position: resets to `(0, 0)`.

This is used at the start of map generation to ensure clean state.

#### Process Character (`_process_char`)

Private helper method that adds a single character's coordinate to the appropriate bucket:

* If character is 'L' (Laro): sets the position and adds to dot coordinates.
* If character exists in the coordinate dictionary: adds the coordinate to that character's set.

This separates the coordinate adding logic from the main scanning loop.

#### Map Coords Generator (`map_coords_generator`)

The primary function for parsing level files and populating coordinate buckets:

1. Calls `_reset_coords()` to clear all existing coordinates.
2. Resets the mushroom counter to 0.
3. Iterates through every cell in `level_as_list`.
4. For each character, calls `_process_char()` to add it to the appropriate coordinate set.
5. If the character is '+' (mushroom), increments the mushroom counter.

This method is called whenever a level is loaded to establish the initial positions of all game objects.

#### Updating Tree Grouper (`updating_tree_grouper`)

Groups adjacent tree coordinates into connected components using flood-fill algorithm:

1. Maintains a `visited` set to track processed trees.
2. Uses an inner recursive function `_tree_grouper()` that:
   * Takes a tree coordinate and a group set.
   * Adds the tree to the group and marks it as visited.
   * Recursively visits all adjacent trees (up, down, left, right).
   * Returns the completed group set.
3. Iterates through all tree coordinates, creating a new group for each unvisited tree.
4. Returns a list of sets, where each set contains all trees in one connected component.

This is important for the flamethrower mechanic, which burns entire connected tree groups.

#### Tile Updater (`tile_updater`)

Updates the level representation when Laro moves:

1. Stores Laro's current position.
2. Uses helper function `_is_in_set()` to determine what tile Laro is currently on (item, water, paved, or empty space).
3. Replaces Laro's old position with the appropriate underlying character.
4. Sets the new position to 'L' (unless it's water '~', in which case water remains visible).
5. Updates Laro's coordinate in the dictionary to the new position.

This ensures the map correctly displays both Laro's new position and what was left behind when Laro moved.

### Player Mechanics `player.py`

This file handles player input processing and movement calculation. It contains the `Player` class that manages action input and coordinate updates.

#### Constructor (`__init__`)

Initializes the `Player` class with a reference to the shared `Coordinates` object, allowing access to Laro's current position.

#### Action Input (`action_input`)

Prompts the user for an action string and returns it in lowercase:

* Displays the prompt `"action:"`.
* Reads user input from stdin.
* Converts to lowercase for case-insensitive controls.
* Returns the processed input string.

This method is only called in interactive mode when the player needs to provide input.

#### Movement (`movement`)

Translates raw input into movement results:

**Parameters:**
* `current_input`: The action string to process (e.g., "wasd").
* `after_game`: Flag indicating if the game has ended (1) or is ongoing (0).

**Returns:**
A tuple of `(new_coords, action, remaining_actions)`:
* `new_coords`: The target coordinate Laro would move to.
* `action`: The single action character being processed.
* `remaining_actions`: Any unprocessed characters in the input string.

**Logic:**
1. Retrieves Laro's current position from coordinates.
2. Validates position (defaults to `(0, 0)` if invalid).
3. Returns current position and empty strings if input is empty.
4. Extracts the first character as `action` and remainder as `remaining_actions`.
5. If action is invalid (not in `directions`), returns current position with empty action.
6. If game is not over (`after_game == 0`):
   * Looks up the direction vector for the action (e.g., 'w' → `(-1, 0)`).
   * Calculates new position by adding direction to current position.
   * Returns the new position, action, and remaining actions.
7. If game is over, returns current position without calculating movement.

This method handles both single actions and strings of actions, processing them one at a time and passing remaining actions back for future processing.

### Inventory Mechanics `inventory.py`

This file manages the player's inventory system using a dataclass. The inventory can hold at most one item at a time (Axe or Flamethrower).

#### Dataclass Structure

The `Inventory` dataclass has three fields:

* `coords`: Reference to the `Coordinates` object to check item positions.
* `comments`: List of strings for displaying messages to the player.
* `inventory`: String indicating current held item ('None', 'Axe', or 'Flamethrower').

#### Pickup Item (`pickup_item`)

Attempts to pick up an item when the player presses 'p':

**Parameters:**
* `action`: The action character (must be 'p' to trigger pickup).
* `tile`: The coordinate tuple where Laro is currently standing.

**Logic:**
1. Only proceeds if `action == 'p'`.
2. Checks if inventory is empty (`inventory == 'None'`).
3. If empty:
   * Checks if tile contains an Axe ('x'):
     * Sets `inventory` to 'Axe'.
     * Removes the axe from the coordinate set.
     * Adds comment "You can now cut a tree".
   * Checks if tile contains a Flamethrower ('*'):
     * Sets `inventory` to 'Flamethrower'.
     * Removes the flamethrower from the coordinate set.
     * Adds comment "You can now burn a forest".
4. If inventory already has an item:
   * Adds comment indicating inventory is full with current item.

Items are consumed on pickup (removed from the map). The player must use the item before picking up another.

#### Use Item (`use_item`)

Currently a placeholder method with no implementation. Items are automatically consumed when used against trees (handled in `tree_mechanics.py`).

### Rock Mechanics `rock_mechanics.py`

This file handles rock pushing mechanics and their interactions with water tiles. Rocks can be pushed into empty spaces or water.

#### Constructor (`__init__`)

Initializes the `RockMechanics` class with:

* `coords`: Reference to the `Coordinates` object for tracking positions.
* `comments`: List for displaying messages to the player.
* `level_as_list`: 2D list representing the current level grid for direct tile updates.

#### Rock Movement (`rock_movement`)

Main method that attempts to move a rock when Laro pushes it:

**Parameters:**
* `action`: The direction action ('w', 'a', 's', 'd').
* `curr_rock`: The coordinate tuple of the rock being pushed.

**Logic:**
1. Stores the current rock position in `self.curr_rock`.
2. Looks up the direction vector from `global_variables.directions`.
3. Calculates `self.next_tile` (where the rock would move to).
4. Uses helper `_is_in_set()` to check tile types.
5. Checks if rock is blocked by:
   * Tree groups.
   * Items (Axe or Flamethrower).
   * Another rock.
   * If blocked: adds comment "Something is blocking the rock!" and returns.
6. If next tile is empty space ('.' or '_'): calls `_rock_to_free_space()`.
7. If next tile is water ('~'): calls `_rock_to_water()`.

#### Rock to Water (`_rock_to_water`)

Private method that handles rock pushed into water:

1. Updates the level grid:
   * Sets next tile to '_' (paved tile).
   * Sets current rock position to '.' (empty space).
2. Updates coordinate sets:
   * Removes next tile from water coordinates ('~').
   * Adds current position to dot coordinates ('.').
   * Removes current position from rock coordinates ('R').
   * Adds next tile to paved coordinates ('_').
3. Adds comment "A paved tile is made!".

This creates a permanent path across water that Laro can walk on.

#### Rock to Free Space (`_rock_to_free_space`)

Private method that handles rock pushed into empty space:

1. Updates the level grid:
   * Sets next tile to 'R' (rock).
   * Sets current rock position to '.' (empty space).
2. Updates coordinate sets:
   * Adds current position to dot coordinates ('.').
   * Removes current position from rock coordinates ('R').
   * Adds next tile to rock coordinates ('R').

The rock simply shifts one position in the pushed direction.

### Tree Mechanics `tree_mechanics.py`

This file manages interactions between the player and trees, including using items (Axe and Flamethrower) to remove tree obstacles.

#### Constructor (`__init__`)

Initializes the `TreeMechanics` class with:

* `inventory`: Reference to the `Inventory` object to check and consume items.
* `coords`: Reference to the `Coordinates` object for tracking tree positions.
* `comments`: List for displaying messages to the player.
* `level_as_list`: 2D list representing the current level grid for direct tile updates.

#### Item Tree Interaction (`item_tree_iteraction`)

Main method that determines if Laro can pass through a tree:

**Parameters:**
* `new_laro_coord`: The coordinate Laro is trying to move to.

**Returns:**
* `True` if the path is cleared (tree removed or no tree present).
* `False` if tree blocks movement.

**Logic:**
1. Initializes `path_cleared` as `False`.
2. Retrieves tree groups from coordinates (handles both set and list formats).
3. For each tree group:
   * Checks if target coordinate is in the group.
   * Attempts to use Axe with `_use_axe()`.
   * If Axe fails, attempts to use Flamethrower with `_use_flamethrower()`.
   * If either succeeds, sets `path_cleared` to `True`.
4. If path is not cleared, adds comment "A tree is blocking your way!".
5. Returns the `path_cleared` status.

This method is called before Laro moves to check if tree obstacles can be removed.

#### Use Axe (`_use_axe`)

Private method that uses the Axe to remove a single tree:

**Parameters:**
* `curr_tree_coord`: The coordinate of the tree to remove.
* `tree_group`: The set of trees in the connected group.

**Returns:**
* `True` if Axe was used successfully.
* `False` if player doesn't have an Axe.

**Logic:**
1. Checks if inventory contains 'Axe'.
2. If yes:
   * Adds tree coordinate to dot coordinates ('.').
   * Removes tree from its group set.
   * Sets inventory to 'None' (consumes the Axe).
   * Adds comment "A tree was chopped!".
   * Returns `True`.
3. If no Axe, returns `False`.

The Axe only removes the single tree Laro is trying to move into.

#### Use Flamethrower (`_use_flamethrower`)

Private method that uses the Flamethrower to burn an entire tree group:

**Parameters:**
* `tree_group`: The set of all trees in the connected group.

**Returns:**
* `True` if Flamethrower was used successfully.
* `False` if player doesn't have a Flamethrower.

**Logic:**
1. Checks if inventory contains 'Flamethrower'.
2. If yes:
   * Iterates through all trees in the group:
     * Sets each tree tile to '.' in the level grid.
     * Adds each coordinate to dot coordinates.
   * Removes the entire tree group from the tree groups list.
   * Sets inventory to 'None' (consumes the Flamethrower).
   * Adds comment "Deforestation!".
   * Returns `True`.
3. If no Flamethrower, returns `False`.

The Flamethrower is more powerful than the Axe, clearing entire connected forest sections at once.

### After Game Mechanics `after_game.py`

This file contains the `AfterGame` class that checks for win and lose conditions during gameplay.

#### Constructor (`__init__`)

Initializes the `AfterGame` class with:

* `coords`: Reference to the `Coordinates` object to check Laro's position.
* `mushroom_eaten`: List containing count of mushrooms collected `[count]`.
* `mushroom_count`: List containing total mushrooms in level `[total]`.
* `comments`: List for displaying messages to the player.

#### After Game Condition (`after_game_condition`)

Checks if the game has ended and returns the result:

**Returns:**
* `1` if game over condition is met (win or lose).
* `0` if game continues.

**Logic:**

1. **Lose Condition - Water:**
   * Retrieves Laro's position, water coordinates, and dot coordinates.
   * Checks if Laro is standing on a water tile ('~').
   * If yes:
     * Resets Laro's position to `(0, 0)`.
     * Removes Laro's coordinate from water set.
     * Adds Laro's coordinate to dot set (water tile returns).
     * Adds comment "Laro is swimming!".
     * Returns `1` (lose).

2. **Win Condition - All Mushrooms:**
   * Checks if `mushroom_eaten[0] == mushroom_count[0]`.
   * If yes:
     * Adds comment "You won!".
     * Returns `1` (win).

3. **Continue Game:**
   * If neither condition is met, returns `0`.

This method is called after each move in the gameplay loop to determine if the level should end. The win condition requires collecting all mushrooms ('+' tiles) in the level.

### Global Variables `global_variables.py`

This file contains global constants and a utility class used throughout the game. It centralizes commonly used data structures and terminal utilities.

#### Directions Dictionary

```python
directions = {'w': (-1, 0), 'a': (0, -1), 's': (1, 0), 'd': (0, 1), 'p': (0, 0), '@': (0, 0), '!': (0, 0)}
```

Maps action characters to their corresponding row/column offset tuples:
* `'w'`: Move up (row -1).
* `'a'`: Move left (column -1).
* `'s'`: Move down (row +1).
* `'d'`: Move right (column +1).
* `'p'`, `'@'`, `'!'`: No movement (0, 0).

This dictionary is used by the `Player` class to calculate new coordinates from actions.

#### Items Dictionary

```python
items = {'x': 'Axe', '*': 'Flamethrower'}
```

Maps ASCII characters to their item names:
* `'x'`: Axe item.
* `'*'`: Flamethrower item.

Used by the interface decorator to display pickup prompts and by inventory system for item identification.

#### Animations Class

Utility class for terminal animations and screen clearing.

**Constructor (`__init__`):**
* No state required, creates an empty instance.

**Clear (`clear`):**
* Clears the terminal screen in a cross-platform manner.
* Uses `cls` command on Windows (`os.name == 'nt'`).
* Uses `clear` command on Unix/Linux/Mac.

**Start Level (`start_level`):**
* Shows a three-stage animation for starting a level.
* Prints "Starting Level {level}." then ".." then "...".
* Uses increasing sleep delays (0.25s, 0.35s, 0.40s).
* Clears screen after animation completes.

**Restart Level (`restart_level`):**
* Clears screen first, then shows restart animation.
* Prints "Restarting Level {level}." then ".." then "...".
* Uses same timing as start level.
* Does not clear screen after (next screen will be printed immediately).

**Exit Level (`exit_level`):**
* Shows animation for returning to main menu.
* Prints "Returning to Main Menu." then ".." then "...".
* Uses same timing pattern.

**Start Game (`start_game`):**
* Shows animation when game first launches.
* Prints "Starting Shroom Raider." then ".." then "...".
* Clears screen after animation completes.

**Exit Game (`exit_game`):**
* Clears screen first, then shows exit animation.
* Prints "Exiting Shroom Raider." then ".." then "...".
* Uses same timing pattern.

**Burning Animation (`burning_animation`):**
* Currently a no-op placeholder that returns immediately.
* Kept for API completeness and potential future implementation.

These animations provide visual feedback and pacing for game state transitions, improving user experience.

### Stage Printer `stage_printer.py`

This file contains functions for rendering the game interface and level display. It uses decorators and emoji mappings to create the visual representation.

#### Interface Decorator (`interface_decorator`)

A decorator factory that creates decorators for printing the game interface:

**Parameters:**
* `in_menu`: Integer flag (0 or 1) indicating if currently in main menu.

**Returns:**
* A decorator function that wraps gameplay methods.

**Decorator Logic:**

1. **Extract Game State:**
   * Casts first argument to `Game` instance.
   * Retrieves current inventory, Laro's coordinates, comments, mushroom counts.
   * Checks if Laro is standing on an item using `next()` and generator expression.

2. **Print Controls:**
   * Always prints movement controls (W/A/S/D) and Reset (!).
   * If in menu: prints "[@] Exit Game".
   * If in level: prints "[@] Main Menu".

3. **Print Level-Specific Info** (if not in menu):
   *
