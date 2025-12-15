# CS1 Adventure Game — Extra Credit Walkthrough

## Table of Contents
- [CS1 Adventure Game — Extra Credit Walkthrough](#cs1-adventure-game--extra-credit-walkthrough)
  - [Table of Contents](#table-of-contents)
  - [1. Project Structure](#1-project-structure)
  - [2. Create the `uniqueGame` folder](#2-create-the-uniquegame-folder)
  - [2a. Folder not showing?](#2a-folder-not-showing)
  - [3. Create the data files](#3-create-the-data-files)
  - [4. Contents of `rooms.txt`](#4-contents-of-roomstxt)
    - [Format](#format)
    - [Rules](#rules)
  - [5. Contents of `items.txt`](#5-contents-of-itemstxt)
    - [Format](#format-1)
    - [Rules](#rules-1)
  - [6. Update `RoomManager.java`](#6-update-roommanagerjava)
  - [7. Run the raw game](#7-run-the-raw-game)
  - [8. Commands and win path](#8-commands-and-win-path)
    - [9. Customize](#9-customize)

## 1. Project Structure
Open the `CS1-Assignment10-12` project in Eclipse. It should match the structure shown in the screenshot.

## 2. Create the `uniqueGame` folder
Right-click the project `CS1-Assignment10-12` → `New` → `Folder` → name it `uniqueGame` → `Finish`. The folder must sit beside `src/`, not inside it. Copy the starter `.jpg` files from `crowtherAdventure/` into `uniqueGame/`.

To proceed, your project structure must look like this:
```text
CS1-Assignment10/
├── src/
│   └── adventure/
│       ├── AdventureProgram.java
│       ├── Room.java
│       └── RoomManager.java (IMPORTANT: there is a new version of this file , see below)
│
├── uniqueGame/    <-- your game folder
│   ├── rooms.txt  <-- we will fill it with unique content, see format below
│   ├── items.txt  <-- we will fill it with unique content, see format below
│   ├── title.jpg  <-- we will copy/paste these .jpg files into this folder for the raw game- when you are making your own, you will put them here
│   ├── room2.jpg  
│   ├── won.jpg    
│   └── lost.jpg   
│
├── crowtherAdventure/      (prior example)
├── simpleGame/             (prior example)
├── AdventureLib.jar
└── assignDeps.jar
```

## 2a. Folder not showing?
Switch to **Project Explorer**: `Window → Show View → Project Explorer`, then repeat step 2 if needed.

## 3. Create the data files
Right-click `uniqueGame` → `New` → `File` → create `rooms.txt` and `items.txt`. 
![Screenshot 2025-12-14 191331](screenshots/Screenshot%202025-12-14%20191331.png)
Add a new file
![Screenshot 2025-12-14 191404](screenshots/Screenshot%202025-12-14%20191404.png)
call it rooms.txt. Do the *same exact process* again to create items.txt.  Now, once you did that, go inside the crowtherAdventure folder to get the starter images copied (title, win, lost, etc.) into `uniqueGame/`.
![Screenshot 2025-12-14 191729](screenshots/Screenshot%202025-12-14%20191729.png)

![Screenshot 2025-12-14 191752](screenshots/Screenshot%202025-12-14%20191752.png)
copy it
![Screenshot 2025-12-14 191812](screenshots/Screenshot%202025-12-14%20191812.png)
paste inside your game folder
![Screenshot 2025-12-14 192351](screenshots/Screenshot%202025-12-14%20192351.png)
now your structure should look like this, with these exact files to start with.


## 4. Contents of `rooms.txt`
Use the starter content below to make the program run, then customize. Keep spacing and ordering intact.

### Format
```text
1
1
title.jpg  
You are in a small lobby. A hallway leads NORTH. A locked door is EAST.  
KEYCARD
NORTH/2
EAST/3/KEYCARD
DOWN/4

2
room2.jpg
You are in the hallway. The lobby is SOUTH.

SOUTH/1

3
won.jpg
You swipe the keycard, the door clicks open, and you escape. You win!

4
lost.jpg
You fall into a padded room labeled "BONK".

FORCED/1
```

### Rules
* **Room ID** on first line. This is a number
* **Image filename** on next line (the .jpg)
* **Description** on next line. NOTE: must be 1 single line 
* **Item ID required to enter** (or blank) on next line.  This is a word that *must* be already in the items.txt file (e.g. KEYCARD)
* **Exits / actions** on next line
One action per line, using this format: ```ACTION_ID/NEXT_ROOM_ID[/ITEM_REQUIRED]``` 

Where:
```ACTION_ID``` → what the player types (e.g. NORTH, EAST)
```NEXT_ROOM_ID``` → the room number this action leads to
```ITEM_REQUIRED``` (optional) → item needed to use this action
(e.g. EAST/3/KEYCARD)


## 5. Contents of `items.txt`
### Format
Use this starter content, then customize later. Keep spacing and blank lines.

```text
KEYCARD
a scratched blue keycard that says "LAB"

SUSHI
a freshly made plate of sushi rolls

FISHFOOD
a tiny packet of fish food
```

### Rules
* **Item ID** on first line 
* **Description** on the next line - it must be 1 line long
* 1 Blank line between items

---
## 6. Update `RoomManager.java`
Replace your existing `src/adventure/RoomManager.java` with:

```java
package adventure;

import common.CS1Reader;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;

/** Manages rooms and loads data. */
public class RoomManager {
    private HashMap<String, Room> rooms;
    private HashMap<String, GameItem> items;
    private String startingRoomId;

    /** Builds from the provided game folder. */
    public RoomManager(String gameFolder) {
        rooms = new HashMap<String, Room>();
        items = new HashMap<String, GameItem>();

        String itemsFilePath = gameFolder + "/items.txt";
        String roomsFilePath = gameFolder + "/rooms.txt";

        loadItems(itemsFilePath); // items first
        loadRooms(roomsFilePath);
    }

    public Room getStartingRoom() {
        return rooms.get(startingRoomId);
    }

    public Room getRoom(String id) {
        return rooms.get(id);
    }

    private void loadItems(String filename) {
        CS1Reader reader = new CS1Reader(filename);
        String itemLine = reader.readLine();
        while (itemLine != null) {
            if (itemLine.length() > 0) {
                String itemId = itemLine.trim();
                String description = reader.readLine();
                GameItem item = new GameItem(itemId, description);
                items.put(itemId, item);
            }
            itemLine = reader.readLine();
        }
        System.out.println("Loaded items: " + items.keySet());
    }

    private void loadRooms(String filename) {
        CS1Reader reader = new CS1Reader(filename);
        startingRoomId = reader.readLine();

        String line = reader.readLine();
        while (line != null) {
            if (line.length() > 0) {
                String roomId = line.trim();
                String imageFile = reader.readLine();
                String description = reader.readLine();

                String itemLine = reader.readLine();
                if (itemLine == null) {
                    itemLine = "";
                }
                itemLine = itemLine.trim();
                List<GameItem> roomItems = new ArrayList<GameItem>();
                if (itemLine.length() > 0) {
                    String[] itemIds = itemLine.split(",");
                    for (int i = 0; i < itemIds.length; i++) {
                        String itemId = itemIds[i].trim();
                        GameItem item = items.get(itemId);
                        if (item != null) {
                            roomItems.add(item);
                        } else {
                            System.out.println("WARNING: Room " + roomId + " references unknown item [" + itemId + "]");
                        }
                    }
                }

                List<GameAction> actions = new ArrayList<GameAction>();
                line = reader.readLine();
                while (line != null && line.trim().length() > 0) {
                    line = line.trim();
                    String[] parts = line.split("/");
                    if (parts.length < 2) {
                        System.out.println("BAD ACTION LINE (missing '/'): [" + line + "]");
                        line = reader.readLine();
                        continue;
                    }

                    String actionId = parts[0];
                    String nextRoom = parts[1];

                    ArrayList<String> requirements = new ArrayList<String>();
                    for (int i = 2; i < parts.length; i++) {
                        requirements.add(parts[i].trim());
                    }

                    actions.add(new GameAction(actionId, nextRoom, requirements));
                    line = reader.readLine();
                }

                Room room = new Room(imageFile, description, roomItems, actions);
                rooms.put(roomId, room);
                line = reader.readLine();
            } else {
                line = reader.readLine();
            }
        }

        System.out.println("Loaded rooms: " + rooms.keySet());
        System.out.println("Starting room: " + startingRoomId);
    }
}
```

## 7. Run the raw game
1. Right-click `AdventureProgram.java` → `Run As` → `Java Application`.
2. When prompted, enter the game folder name.  

## 8. Commands and win path
Valid commands:
```
NORTH
SOUTH
EAST
DOWN
GET KEYCARD
LIST
QUIT
```
Winning path for the starter content:
```
GET KEYCARD
EAST
```
If an item says "not here": move first, then try again.


### 9. Customize
   I recommend you test the raw game first, then you modify `rooms.txt` and `items.txt` using those file rules to create your own unique adventure.
   Play with it a bunch to get the hang of it. Find images online to use as room images, and place them in the `uniqueGame/` folder. Then edit `rooms.txt`, `items.txt`, and swap in your own images (keep filenames in sync)
    (e.g room2 might become forest.jpg if you want a forest scene).
    
    Note:  Read through the assignment pdf, and remember that you'll need:
        - at least 3 rooms, 
        - 1 item, 
        - 1 transition that requires an item, and 
        - 1 transition that doesn’t require an item
