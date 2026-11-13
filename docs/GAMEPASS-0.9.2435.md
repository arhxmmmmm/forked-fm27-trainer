# Game Pass / Microsoft Store build 0.9.2435.0: notes

Findings on the Microsoft Store (Xbox Game Pass) build of FM27 0.9.2435.0, which the upstream trainer (0.2.3) did not
recognise. Measured on one machine; treat the offsets as observations, not guarantees.

- `game_plugin.dll` build id: `PE-6AE8CF77-17969000` (link timestamp `0x6AE8CF77`, SizeOfImage `0x17969000`).
  The game runs from `C:\Program Files\WindowsApps\SportsInteractive.FootballManager27_0.9.2435.0_x64__5w3tn6tb6stnm\`.
- There is no address profile for this build, so the staff list is unknown and the RTTI fallback for
  `db::HUMAN_NON_PLAYER` finds nothing (that class has no vtable at offset 0 here: instances start with a vbptr).

## Changes

1. **Manager lookup** (`agent/src/game.rs`, `human_via_manager_list`). `HUMAN_NON_PLAYER_MANAGER` is a plain object with a
   vtable at offset 0. It keeps a list (begin, end, cap) of the human managers at `+0x18`. Each list element is tried
   with the existing `check_chain`. The old layout still holds from there: contract at `+0x4E8`, identity block at `+0x440`.
2. **Training calendar** (`week_ok` in `agent/src/game.rs`). The first word of each day block was required to be small
   (`<= 0xFF`); on this build it can hold values like 49920 in a week of rest days. The app never writes that word, so
   the check was dropped. The week list (13 weeks, Mondays, rest code 300) is otherwise accepted as before.
3. **Gauge reading** (`find_pies` in `app/src/vision.rs`). This build draws empty pie segments mid-grey instead of near-white.
   The strict pass used to return whatever it found first: the coloured half of a pie (reading 100 %), or the white
   icon of the neighbouring label (reading 0 %). Candidates must now be a whole, round gauge of the right size, from
   either pass. Regression test: `pies_with_grey_empties_and_neighbouring_icon`.
4. **Auto-read** (`app/src/gui.rs`). Optional switch on the Overview page: re-reads the Breakdown every N seconds
   (5 to 120, default 10) while it is open in FM. Quiet when nothing changed or the Breakdown is not showing, and paused in
   manual-edit mode. Off by default.

## Not done

- Buying and selling players (needs the club list address for this build).
- The in-game date can be missing or imprecise (also a per-build address).
- Tactical Effectiveness is still read from the screen, not from memory.
- The Windows `.exe` attached to releases was cross-compiled with MinGW (`x86_64-pc-windows-gnu`), not MSVC.
