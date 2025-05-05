# Dragon-Ball-Tap-Battle-Decompilation
Sketchware Pro Project


# Technical Wiki - TCBManajer.java

## Index
1. [General Overview](#general-overview)
2. [Game States](#game-states)
3. [Files and Resources](#files-and-resources)
4. [Touch and Button System](#touch-and-button-system)
5. [Important Global Variables](#important-global-variables)

---

## General Overview
The `TCBManajer` class is the core of the engine. It coordinates:

- Loading of characters, cards, HUDs, etc.
- State and transition management
- Reading and writing `.pac`, `.bin`, `.dac` files
- Input and touch system
- Script execution (e.g., `Run()`)

---

## Game States
Controlled by:
- `tcbNow.md` (current mode)
- `tcbNow.lp` (time in mode)

---

## System States (`tcbNow.md`)

`tcbNow.md` represents the current "mode" or phase of real-time execution. It changes frame by frame as the game progresses.

| Value Meaning                             Situation                                                           
|-------|-------------------------------------|----------------------------------------------------------------------|
| 0     Start                                Default initial state                                               
| 71    Error (smap)                        Returns to menu                                                     
| 84    Error (Download/Connection)       Shows warning                                                       
| 87    End of data loading                                                                                     
| 89    Character version check                                                                                 
| 97    Wipe out                                                                                                
| 106   Error or fallback                    Used for reset or failure                                           
| 109   Preparing to download character/cards                                                                     
| 110   Draws character artwork                                                                                 
| 138   Panel creation for preview                                                                              
| 145   Panel                               Mission/panel mode                                                  
| 193   Character selection                 Loading and choosing character                                      
| 196   Post-fight transition                                                                                   
| 197   Final fade before fight starts                                                                          
| 198   Battle initialization                  Initial setup of variables                                          
| 249   Main screen after fight                                                                                 
| 262,263,264,267   Main Menu (button images)                                                                               
| 264   Main Menu (touch collision)                                                                             
| 270   Main Menu (Button String)                                                                               
| 272   Main Menu (redirection)                                                                                 
| 273-276,1015 SMAP                       SMAP visualization                                                  
| 366   Scene effects finalization                                                                              
| 367-373 Ranking screen                                                                                        
| 384   Menu (difficulty selection images)                                                                      
| 390   Battle (active fight)                 Main combat loop                                                    
| 714   Loading                             Loading screen                                                      
| 720   Pre-battle                           Initial fight setup                                                 
| 721   Player initialization                 Sets state and loads characters                                     
| 722   Phase transition                    Used between fights or scenes                                       
| 783-784 Final data loading process                                                                            
| 809   Result screen                       Displays results after battle                                       
| 952   Training Execution                  Training logic ongoing                                              
| 953   AI Execution in training             Automated opponent actions in training                             
| 1006  Restart Battle
| 1008-1013,1016 Character selection screen                                                                     
| 1014  Settings                                                                                                

---

### 1. LoadData[] (indices)
Central vector for loading characters, cards, HUDs, scenarios, etc.

| Index Use                                Notes                         
|-------|------------------------------------|--------------------------------|
| 1-2     Scenario ID                        E.g., 1 = Namek, 2 = Arena    
| 3     Player 1 character                 Reflected in [27]            
| 4     Player 2 character                 Reflected in [28]            
| 5     Loading type                       Can be 0, 1 or 2              
| 6–13  Player 1 cards                    Must be added to 255 for index|
| 14–21 Player 2 cards                    Same scheme as player 1       
| 22–25 Card states                       Like abilities or special effects|
| 27    Player 1 backup character         Helps in transitions/results  
| 28    Player 2 backup character         Same purpose                  
| 29–30 Internal reset                      Reset in `SetLoad()`          

---

### 2. PracticeSetting[]

Controls opponent behavior in training mode (`iPlayMode == 2`).

| Index Function                         Common Values             
|-------|----------------------------------|----------------------------|
| 0     Opponent initial direction       0 = Right, 1 = Left       
| 1     Initial position                 0 = Normal, 1 = Center, etc|
| 2     AI type                          0 = Idle, 1+ = Reacts     
| 3     AI subcommand                    Depends on AI type        
| 4     Reaction to attack               0 = Nothing, 1 = Defend   
| 5     Reserved                         Usually 0                 

---

### 3. PlayerType[]

Defines active characters in the game.

| Index Player     Value           
|-------|------------|------------------|
| 0     Player 1   Character ID    
| 1     Player 2   Character ID    

Mirrored in `LoadData[3]`, `[4]`, `[27]`, `[28]`.

---

### 4. ConfigData[]
- Structure: `ConfigData[char * 100 + 30 + offset]`
- Common Offsets:
  - `+1`: Unlocked
  - `+2`: Downloaded
  - `+3`: Version
  - `+22`: Equipped cards
  - `+53`: XP
  - `+85`: Available?

---

### 5. bCharIndex[], cstatusCount
- `bCharIndex[index] = (byte)index` char index
- `cstatusCount` char count

Example - Tutorial Mode:
- `bCharIndex[0]` = 0
- `cstatusCount` = 1
- Only char00 in characterselect

---

### 6. iPlayMode Values and Functionality

| Value Name                Confirmed Function                             HP Reduction Notes                              
|-------|---------------------|------------------------------------------------|--------------|-------------------------------------|
| 0     Arcade               Story/Arcade mode with rounds                 Yes          Mission with AI                    
| 1     (Unknown)           [Reserved]                                        ?            Not clearly used in code           
| 2     Training              Free training with AI, infinite life                 No           `PracticeSetting` defines AI       
| 3     (Unknown)           [Reserved]                                        ?            Not clearly used in code           
| 4     (Unknown)           [Reserved]                                        ?            Not clearly used in code           
| 5     Ranking              Score challenge mode                           N/A          Loads specific assets              
| 6     SMAP                SMAP viewer                                     N/A          Calls `glview.SmapStart()`         
| 7     Data / Cards         Access to card/config menus                   N/A          Allows deck building               
| 8     Versus (Multiplayer)  Local PvP                                        Yes          Ideal for 2-player mode            
| 9     Versus                Local PvP                                        Yes          Same as above                      

These values are assigned to the `iPlayMode` variable to determine the current game behavior. Each mode changes life system, AI, HUD, and resource loading.

---

## Detailed Important Variables

### CPULevel[]
- **Type**: `int[2]`
- **Description**: Defines AI difficulty (player 2)
- **Possible Values**:
  - `0-7`: Used in training mode
- **Usage**: Affects `CPUCommand`, `CPUCount`, response and defense

---

### CreatePanel(...) (function)
- **Description**: Responsible for drawing UI/panel interfaces
- **Common Parameters**:
  - `GlobalWork gw`
  ?
  - `actype`: 0 = common, 5 = demo_00 and effect, 1 = select0, 6 and 7 = chardemo
  - `ano`
  ?
  - `wactflag`
  - `x`, `y`
  
BUTTON MAPPING (slightly inaccurate)
start of line x: ~ -60
start of line y: ~ - 20
end of line x: ~ 540
end of line y: ~ 320

- **Usage**: Menu, HUD, card selection, results

---

## Files and Resources
- Read by `SetBinary()`, `readDataFile()`, `Utility.checkFile()`
- Examples: `char01.pac`, `card001.pac`, `save.bin`
- Some are `.dac`, `.bin`, or `.pac`
- `pack_unpack()` used to install compressed files

---

## Touch and Button System

### Checks:
- `TapType[]`, `TapXPos[]`, `TapYPos[]` — touch input
- `_commandButton[]`, `_commandButtonBuf[]` — buttons and commands

---

### AddPad:
- `int AddPad(int x, int y, int width, int height, int type, int keyCode)`
Type 2, 3, 5, 6, 7: `PosNow[0-1/x-y] = -100`

---

### PadId[]:
- 0: Joystick 
- 1: Buttons
- 4: Char Select
- 7: market button (character select)

---

## Important Global Variables

### Control and System
- `tcbNow`: current state control
- `gw`: pointer to `GlobalWork`
- `iStage`, `iPlayMode`, `iStagePoint`: stage control
- `bDrawLoading`: if loading screen is active

### Players
- `PlayerType[]`: current characters
- `PlayerCard[][]`: player cards
- `PlayerLife[]`, `PlayerState[]`: life and status
- `PlayerXPos[]`, `PlayerYPos[]`: screen position
- `PlayerEventFlag[]`: active events

### Cards
- `LoadData[6-21]`: loaded cards
- `PlayerCardCount[]`: number of cards
- `PlayerCardFormation[][]`: active formation

### Input System
- `TapType[]`, `TapXPos[]`, `TapYPos[]`: touches
- `TouchesCommand[][]`: commands
- `TouchesVal1/2Buf`, `TouchesVal1/2Work`: analysis values

### Button Control
- `_commandButton[]`: pressed buttons
- `_commandButtonBuf[][]`: input buffer
- `_commandButtonWork[][]`: state control

### Others
- `iBackNo`, `iBackNo2`: background stage
- `bTaskRepeat`: repeat frame

---