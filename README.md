### Chessnut Move Chess Board API Documentation

#### **1. Bluetooth Connection**
- **Device Name**: "Chessnut (*)"
  - including Air, Air+, Go, Pro
- **FEN Notification Service**:  
  - UUID: `1b7e8261-2877-41c3-b46e-cf057c562023`  
  - Characteristic (FEN data notification): `1b7e8262-2877-41c3-b46e-cf057c562023`  
- **Operation Commands Service**:  
  - UUID: `1b7e8271-2877-41c3-b46e-cf057c562023`  
  - Characteristic (Write commands): `1b7e8272-2877-41c3-b46e-cf057c562023`  
  - Characteristic (Response notifications): `1b7e8273-2877-41c3-b46e-cf057c562023`  

  > **Notice:** If you cannot receive the full FEN data, please set the BLE MTU to 500 after the BLE connection is established.
---

#### **2. Obtaining FEN (Board Position)**
To enable the real time FEN report, send `[0x21, 0x01, 0x00]` to the Operation Write Characteristic.
After enabling notifications on the FEN characteristic, a **36-byte array** is received.  
- **Position Data**: Bytes `[2]` to `[33]` (32 bytes total) represent board state.  
- **Encoding**:  
  - Each byte encodes **two squares** (4 bits per square).  
  - **Square Order**: Starts at `h8` → `g8` → `f8` → ... → `a8` → `h7` → ... → `a1`.  
  - **Lower 4 bits** of a byte = First square (e.g., `h8`).  
  - **Higher 4 bits** = Next square (e.g., `g8`).  
- **Piece Mapping**:  
  ```javascript
  const pieces = ['', 'q', 'k', 'b', 'p', 'n', 'R', 'P', 'r', 'B', 'N', 'Q', 'K']; 
  // 0: Empty, 1: Black Queen, 2: Black King, ..., 12: White King.
  ```  
- **FEN Conversion (JavaScript Example)**:  
  ```javascript
  const pieces = ['', 'q', 'k', 'b', 'p', 'n', 'R', 'P', 'r', 'B', 'N', 'Q', 'K'];
  let fen = "";
  let empty = 0;
  for (let row = 0; row < 8; row++) {
    for (let col = 7; col >= 0; col--) {
      const index = Math.floor((row * 8 + col) / 2) + 2;
      const pieceVal = (col % 2 === 0) 
          ? data[index] & 0x0F 
          : data[index] >> 4;
      const piece = pieces[pieceVal];
      if (piece === '') empty++;
      else {
        if (empty > 0) fen += empty;
        fen += piece;
        empty = 0;
      }
    }
    if (empty > 0) fen += empty;
    if (row < 7) fen += '/';
    empty = 0;
  }
  ```

---

#### **3. Setting LEDs**
Send a **34-byte command** to the Operation Write Characteristic:  
- **Command Structure**:  
  ```[0x0a, 0x08, 0, 0, 0, 0, 0, 0, 0, 0]```  
- **LED Encoding** (Bytes `[2]` to `[9]`):  
  - Each byte controls 8 squares, index from 8 to 1, h to a.  
  - **Bit Values**:  
    - `0`: LED off  
    - `1`: LED on  

---

#### **4. Reading Battery Level**
- **Command** (Write Characteristic):  
  `[0x29, 0x01, 0x00]` (3 bytes).  
- **Response** (Notification Characteristic):  
  `[0x2a, 0x02, batteryLevel, reserved]` 
  - `batteryLevel.bit[7]`: `1`: Charging, `0`: Not charging.
  - `batteryLevel.bit[0..6]`: `0`–`100` (percentage).  

---
