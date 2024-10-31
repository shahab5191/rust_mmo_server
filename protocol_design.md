# Communication Protocol

## Request Structure:

Meta data
- Header_size: 1 bytes
- Body_size: 1 bytes
Header
- request_id: 2 bytes
- session_id: 4 bytes
body starts here
- body structure depends on request

## Requests
### Client Requests:
1. **Spells**
2. **Movement**
3. **Character Information**
4. **Entity Information** (Players, NPCs, Items)
5. **Environment Interactions** (e.g., Pick up item, interact with NPC)
6. **Chat and Social Actions** (Chat messages, Friend requests)

---

### **1. Spells**
**Purpose**: To cast spells or use abilities, specifying the spell ID and target.

```plaintext
spell_id: 2 bytes
cast_start_time: 4 bytes  (unix time)
target:
  target_type: 1 byte (self, NPC, player, area)
  if target_type is area:
    target_position: x: 2 bytes, y: 2 bytes (relative position to caster)
  else:
    target_id: 4 bytes (NPC-ID, player-ID, or null for area-based spells)
```
---

### **2. Movement**
**Purpose**: To communicate basic movement actions, including position and movement type. Server will handle specific speed adjustments.

```plaintext
movement_type: 1 byte (0 = walk, 1 = run)
direction: 3 floats (x, y, z)
jump: 1 boolean
```
---

### **3. Character Information**
**Purpose**: To request different sets of data about the client’s character.

```plaintext
info_type: 2 bytes
	- 0x01 = Inventory
	- 0x02 = Equipped items
	- 0x03 = Titles
	- 0x04 = Name and general stats
	- 0x05 = Social group information (friends, guild)
	- 0x06 = Position
	- 0x07 = Quest log
```

---

### **4. Entity Information**
**Purpose**: To request data about entities in the game world (players, NPCs, items).

#### **a. Players**
```plaintext
player_id: 4 bytes
info_type: 1 byte
	- 0x00 = General (name, class, buffs)
	- 0x01 = Detailed (equipment, talents, stats)
	- 0x02 = Position and status
```

#### **b. NPCs**
```plaintext
npc_id: 4 bytes
info_type: 1 byte
	- 0x00 = Basic (name, level, faction)
	- 0x01 = Detailed (stats, quest interaction data)
```

#### **c. Items**
```plaintext
item_id: 4 bytes
info_type: 1 byte
	- 0x00 = Basic (name, type)
	- 0x01 = Detailed (effects, stats)
```

---

### **5. Environment Interactions**
**Purpose**: To handle player interactions with the environment, such as interacting with items or interacting with NPCs.

##### Request
```plaintext
interaction_type: 1 byte 
    - 0x0 = Interact with item
    - 0x1 = Interact with NPC
target_id: 4 bytes (ID of item or NPC)
```
##### Response
```plaintext
if interaction contains dialogue, sends back open dialogue command with details:
    - dialogue_open_command: 0x10 2 bytes
    - dialogue_id: 4 bytes
```

**Example Enhancements**:
- Use additional fields like `npc_dialogue_option` to select specific NPC options.

---

### **6. Chat and Social Actions**
**Purpose**: To send chat messages or social actions like friend requests.

#### **a. Chat**
```plaintext
channel: 1 byte (0 = General, 1 = Party, 2 = Guild, etc.)
message_length: 1 byte
message: variable length (limited by `message_length`)
```

#### **b. Social Actions**
```plaintext
action_type: 1 byte (0 = Friend request, 1 = Accept friend request, 2 = Block)
target_player_id: 4 bytes
```

---

### Other Suggestions
- **Request IDs**: Include a unique request ID for each client request (e.g., 2 bytes), which can help the server validate or track asynchronous responses.
- **Error Handling**: Add a protocol for error responses, where the server can respond with error codes for invalid or unsupported requests.
- **Heartbeat**: Include a simple “heartbeat” message from client to server to maintain connection and detect disconnects or client-side lags.
