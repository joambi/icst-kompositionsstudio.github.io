# AmbiEncoder64 Motion Map

REAPER Lua tool for writing ICST `AmbiEncoder_64` XYZ automation from per-source
motion maps.

## What It Does

- Assigns motion shapes per source `S0` to `S63`
- Writes ICST `AmbiEncoder_64` XYZ FX automation
- Scales all movement to the current REAPER time selection
- Creates or overwrites a render region for B-format exports
- Supports source subsets, e.g. only `S0` and `S9`
- Supports optional Z-axis motion

## Scripts

- `Scripts/JS_AmbiEncoder64_Motion_Map_GUI.lua`
  - Main user-facing GUI
  - Select sources and motion shapes
  - Writes automation plus region via the writer script
- `Scripts/JS_Write_AmbiEncoder64_Spat_Motion_Automation.lua`
  - Writer engine
  - Can also be used directly with a text dialog
- `Scripts/JS_Normalize_Selected_BFormat_Track_To_-23_LUFS.lua`
  - Analyzes a selected/only 4ch B-format media item with `ffmpeg`
  - Applies the required gain to the selected track volume for `-23 LUFS`
- `Scripts/JS_Analyze_Selected_BFormat_Direction.lua`
  - Analyzes dominant B-format XYZ direction over time
  - Writes CSV data with project time, direction vector, azimuth, elevation, and energy
- `Scripts/JS_Draw_BFormat_Direction_Score_From_CSV.lua`
  - Converts a direction-analysis CSV into a visual SVG score
  - Draws X/Y movement plus azimuth, elevation, and energy timelines
- `Scripts/JS_Write_BFormat_Direction_Timeline_From_CSV.lua`
  - Converts the direction-analysis CSV into synchronized REAPER empty items
  - Creates separate score tracks for direction, elevation, and energy

## Install In REAPER

1. Open `Actions > Show action list`
2. Click `New Action > Load ReaScript`
3. Load both scripts from this folder:
   - `Scripts/JS_AmbiEncoder64_Motion_Map_GUI.lua`
   - `Scripts/JS_Write_AmbiEncoder64_Spat_Motion_Automation.lua`
4. Run `AmbiEncoder64 Motion Map GUI`

The GUI expects the writer script to live in the same folder.

## Workflow

1. Select the REAPER track that contains `VST3: AmbiEncoder_64 (ICST)`
2. Set a time selection
3. Open `AmbiEncoder64 Motion Map GUI`
4. Enable the desired sources
5. Pick one motion shape per enabled source
6. Adjust `Steps/sec`, spreads, `Motion amount`, and `Use Z motion`
7. Click `Write Automation + Region`
8. Render the created region as B-format

## B-format LUFS Normalize

Use `JS_Normalize_Selected_BFormat_Track_To_-23_LUFS.lua` after rendering a
4-channel B-format item.

1. Select exactly one REAPER track
2. Select the rendered 4ch B-format item on that track
3. Run the LUFS normalize script

The script analyzes the selected item segment and changes the selected track
volume so playback lands at `-23 LUFS`. It expects `ffmpeg` at
`/opt/homebrew/bin/ffmpeg`.

## B-format Direction Analysis

Use `JS_Analyze_Selected_BFormat_Direction.lua` on a rendered 4-channel B-format
item to inspect the dominant direction inside the recording.

The script writes a CSV with:

```text
index,time_project_s,time_item_s,x,y,z,azimuth_deg,elevation_deg,energy_db,active
```

Input options:

- `Order WXYZ/AmbiX`: use `WXYZ` for FuMa/WXYZ, `AmbiX` for ACN/SN3D
- `Window ms`: analysis resolution, e.g. `100`
- `Energy gate dB`: low-energy windows are marked inactive
- `Write markers`: optional REAPER markers with azimuth/elevation
- `CSV filename`: relative to the project folder, or an absolute path

The default CSV filename is derived from the region under the B-format item.
If no region exists there, the script uses the B-format media filename.

## B-format Direction Score

Use `JS_Draw_BFormat_Direction_Score_From_CSV.lua` after the direction analysis.

1. Run `JS_Analyze_Selected_BFormat_Direction.lua`
2. Run `JS_Draw_BFormat_Direction_Score_From_CSV.lua`
3. Use the same CSV filename, e.g. `BFormat_direction_analysis.csv`
4. Open the generated SVG score in a browser or vector editor

The score contains:

- Top-view X/Y movement path
- Start/end markers
- Time-colored movement points
- Azimuth timeline
- Elevation timeline
- Energy timeline

## B-format Timeline Score

Use `JS_Write_BFormat_Direction_Timeline_From_CSV.lua` when the visual score
should run directly inside REAPER alongside the audio.

The script detects the active region at the play cursor, time selection start,
or edit cursor. If a region is found, the default CSV name is derived from that
region, and only CSV rows inside that region are written. The score tracks are
inserted directly below the selected track.

The script creates three item tracks:

- `BDir Score Direction`: front/left/back/right direction blocks
- `BDir Score Elevation`: up/mid/down blocks
- `BDir Score Energy`: low/medium/high/silent energy blocks

Input options:

- `CSV filename`: direction-analysis CSV
- `Min item ms`: minimum score item length, e.g. `500`
- `Clear old`: remove previous score tracks with the same prefix
- `Track prefix`: default `BDir Score`
- `Energy gate dB`: ignores weak direction windows below this level

## Source Mapping

The GUI labels sources as `S0` to `S63`.

Internally this maps to ICST/OSC source blocks `1` to `64`:

```text
GUI S0  -> ICST Source 1 -> X/Y/Z parameter block 11/12/13
GUI S1  -> ICST Source 2 -> X/Y/Z parameter block 16/17/18
GUI S2  -> ICST Source 3 -> X/Y/Z parameter block 21/22/23
...
GUI S63 -> ICST Source 64
```

## Motion Shapes

- `Line`
- `Arc+`
- `Arc-`
- `S`
- `Step`
- `Zig`
- `Circ`
- `Spir`
- `Four`
- `Heart`
- `Card`
- `Rose8`
- `Bern`
- `Astr`
- `Epi`
- `Lis`

`Motion amount` scales the shape around its center. Circle and spiral use a
shared X/Y radius so they stay round instead of becoming ovals.
`Four` combines several harmonics into one denser XYZ path.
The newer mathematical shapes are planar curves with optional Z modulation.

## Release Checklist

- Verify with a small source set, e.g. `S0`, `S1`, `S9`
- Verify all 64 sources write the correct ICST parameter blocks
- Verify `Use Z motion` on/off
- Verify overwrite region behavior
- Verify B-format region render
- Add screenshots or a short GIF for GitHub
- Tag a first release once the workflow is stable
