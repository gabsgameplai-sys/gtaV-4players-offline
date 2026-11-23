# gtaV-4players-offline

A "patch" for the **Offline Coop - SplitScreen** mod that makes it compatible with **3 and 4 players**, with real 2x2 viewports and separate controls.

Credit goes to AlexDevilish and opencode.

Original mod link: https://www.nexusmods.com/gta5/mods/864?tab=files

This mod is **NOT affiliated with Rockstar / Take-Two**. Use at your own risk and always back up your files.

---

## What it does

- Extends the original 2-player offline co-op to **3 or 4 players**
- Renders **real viewports in a 2x2 grid** (no automatic viewport switching)
- Independent controllers for every player (gamepads + keyboard fallback)
- Menu to change characters, models and who the keyboard controls

| File | Description |
|------|-------------|
| `OfflineCoopMod.asi` | Native engine rewritten in C++ (Zig). D3D11 hook composes up to 4 real viewports. |
| `scripts/OfflineCoop.dll` | C# extension: spawn/control of P3 and P4, cameras, gamepads and menus. |
| `scripts/OfflineCoop.ini` | Configuration (`NumPlayers = 3` or `4`). |
| `XInputBlock.asi` | **Must be the original version**. Blocks the P2 slot so GTA does not steal the controller. |

---

## Dependencies

- **ScriptHookV**
- **ScriptHookVDotNet** (the v2 API script `ScriptHookVDotNet2.dll` is required)
- **NativeUI**
- **SharpDX** + **SharpDX.DirectInput** (needed for the extra gamepads)
- **XInputBlock** (original version only — the custom 4-slot build crashes on startup)

Compatible versions used here: GTA `1.0.3889.0` with ScriptHookV `3889.0.1158.13`.

All of them are already included in the `deps/` folder of this package.

---

## Installation

1. Install the dependencies above (or just copy the contents of `deps/`).
2. Copy these files to the **root** of your GTA V folder:
   ```
   dinput8.dll
   ScriptHookV.dll
   xinput1_4.dll
   ScriptHookVDotNet-Loader.asi
   ScriptHookVDotNet.dll
   ScriptHookVDotNet2.dll
   ScriptHookVDotNet3.dll
   ScriptHookVDotNet.ini
   XInputBlock.asi
   OfflineCoopMod.asi
   ```
3. Copy these files to the `scripts/` folder (create it if it does not exist):
   ```
   NativeUI.dll
   SharpDX.dll
   SharpDX.DirectInput.dll
   OfflineCoop.dll
   OfflineCoop.ini
   ```
4. Edit `scripts/OfflineCoop.ini` and set:
   ```ini
   [Options]
   NumPlayers = 3
   ```
   Use `3` for three players or `4` for the full 2x2 grid.
5. Start the game with the controllers already connected.

---

## Controllers

At startup the gamepads are assigned by physical order:

| Gamepad | Player |
|---------|--------|
| 1st | P1 (game, native XInput) |
| 2nd | P2 (mod, DirectInput) |
| 3rd | P3 (mod, DirectInput) |
| 4th | P4 (mod, DirectInput) |

The P1 controller is never touched by the mod, so one pad never moves two players. If the automatic order does not work for you, use **Select Controller** in the menu (PgUp).

### Extra gamepad buttons (P3/P4)

- Left stick / DPad → move (relative to the camera)
- **A** → jump
- **B** → enter/exit vehicle
- **X / Y** → shoot
- **RB** → sprint

### Keyboard fallback

- P3: `U/H/Y/J` move, `I` jump, `O` vehicle, `P` shoot
- P4: arrows move, `Right Ctrl` jump, `Right Shift` vehicle, `Apps` shoot
- `F6` → legacy manual viewport cycle

---

## Menu (PgUp)

Press **PageUp** to open the menu. Besides the original options, there is a submenu:

### Trocar Personagem / Change Character

| Option | Function |
|--------|----------|
| Jogador | Choose P1, P2 or P3 as the target |
| Trocar modelo (aleatório) | Change the target's model to a random one |
| Digitar nome do modelo | Type a model name, e.g. `player_zero` |
| Girar peds P1<-P2<-P3<-P1 | Rotate the characters (requires all 3 spawned) |
| Teclado controla | Choose which player receives `U/H/Y/J` |

---

## Technical notes

The C# script switches the rendered camera every frame and writes the selector into the shared memory `GTASplitScreenCoop` (12 bytes: `+0` active, `+4` selector 1..4, `+8` frame count). The `.asi` reads it in the D3D11 `Present` hook, snapshots each camera and composites the quadrants onto the backbuffer.

Logs are written to `%TEMP%`:

- `asi4p.log` → viewport engine (`hook installed`, `active signal received`, `textures created`, `selector=1/2/3/4`)
- `XInputBlock.log` → gamepad blocking

---

## Building from source

Sources are in `src/`:

- `src/OfflineCoopSrc` → C# project (.NET Framework 4.8). Adjust the `HintPath` entries in the `.csproj` to your GTA folder and run `dotnet build -c Release`.
- `src/asi4p.cpp` → native engine. Example (Zig 0.9.1):
  ```
  zig c++ -target x86_64-windows-gnu -shared -O2 -o OfflineCoopMod.asi asi4p.cpp -lkernel32 -luser32
  ```
- `src/xinputblock4p.cpp` → **experimental**, do not use in production (crashes on startup).
