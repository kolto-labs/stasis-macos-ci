Stasis 0.0.1b
by Kolto Labs
https://koltolabs.bocloud.workers.dev/

README_REALLY.txt. Stasis was formerly KPTAS.

Yes, really. This is the only doc that ships, so it's the one to read.


KNOWN ISSUES IN 0.0.1b
---------------------

Read these first. They're real, I know about them, and they're next on the
list.

- The Dantooine Ruins AutoSave can crash the game. It happens when that
  AutoSave drops you straight into the Ruins while Bastila is out of your
  party. The game opens the party screen before your character has finished
  loading, and down it goes. Don't start a recording from that AutoSave.
  Start from the save before it.
- A save made after the Routine can fail to load later with "load error 4"
  (the area data is missing). Right after the Routine, load your new save
  once before you keep going. If it fails with load error 4, redo the Routine
  from an earlier point. A fix is being tested for the next version.
- Save tricks aren't reliable yet. That covers save_ms, @save and
  !altf4@save. They're described below because the code has them, but don't
  build a route on them in this version.
- Authoring runs at about 25 frames per second, slower than the game's real
  speed. Turbo (Ctrl+T) helps playback. Nothing in 0.0.1b runs at full speed.
- Linux and Wine only. Tested on wine-11.0 Staging. Wine needs 32-bit
  support (wow64), because swkotor.exe is a 32-bit program.
- KotOR 1, GOG version 1.03 only. ./stasis check tells you if your exe is the
  wrong one.
- Disk. Every take has its own storage budget, and the sessions I measured
  used about 55 to 83 MB each. Lots of rewinds means lots of takes. Watch
  your free space.
- The live score keys (Ctrl+Alt+P, Ctrl+Alt+U, Ctrl+Alt+O) only work while
  the Stasis window has focus, not the game window.


WHAT IT IS
----------

Stasis is a frame-perfect TAS tool for KotOR 1. It isn't an emulator and it
isn't a replay mod. It runs the real swkotor.exe under Wine and holds the game
still between frames. Nothing moves until you tell it to.

How: a small DLL (dinput8.dll) loads into the game before anything else does.
It sits at the top of the game's main loop and stops there every pass. Stasis
on the other side decides what that frame gets: which keys are held, where the
mouse is, what time it is. The game's clock is fed by Stasis, not by your PC.
So frame 4000 is frame 4000 no matter how slow your machine got on the way
there.

What you get out of that:

- Every frame is recorded, and you can rewind to any of them.
- Rewinding doesn't overwrite anything. Play on from an old frame and you get
  a new take. The old one stays on disk.
- The same inputs give the same game. A recording replays the same way every
  time.

The honest version of that last line: it has passed strict restore and replay
checks on my K1 GOG setup under Wine. That's not a promise for every save,
machine or mod. If you find a frame where it doesn't hold, that's a bug, and I
want it (see REPORTING A PROBLEM).

In 0.0.1b the game takes its input from Stasis only: the piano roll, Apply, a
loaded score. Your real keyboard and mouse don't drive the game while it runs
under Stasis.

You'll still see old names in places: "Telos" in some console lines, "kptas"
in folder names. Same tool.


WHAT YOU NEED
-------------

- Linux, with Wine (32-bit support included).
- KotOR 1, GOG version 1.03 (swkotor.exe). Steam, disc, a patched exe or K2
  won't run.
- Python 3.11 or newer, with PySide6 installed. Your package manager calls
  it python3-pyside6, pip calls it PySide6.
- setarch (part of util-linux). The example config uses it to turn off
  address randomization so runs repeat.
- Free disk space. ./stasis check refuses under 2 GB free and warns under
  10 GB.
- NO KPM. If KotOR Patch Manager is installed in the game folder, Stasis
  refuses to start. A vulnerability was found in KPM, and I'm not running a
  tool inside a process that loaded it. See TROUBLESHOOTING for how to get the
  original files back.


SETTING IT UP
-------------

1. Unpack it. You get:

     stasis-<version>/
       stasis               the launcher
       stasis.toml.example  the config you copy
       lib/                 Stasis libraries and the default layout
       shim/dinput8.dll     the DLL that goes into the game
       VERSION              version and the shim's checksum
       MANIFEST.sha256      checksums for everything above
       README_REALLY.txt    this

2. Copy the config and edit the copy:

     cp stasis.toml.example stasis.toml

3. In stasis.toml, set:

     [run]  out_dir     where recordings go. The example says
                        ~/stasis/recordings, and Stasis creates it
     [game] dir         the folder with swkotor.exe in it. Use a copy
                        of the game with no KPM
     [game] wineprefix  your Wine prefix. Leave it commented out and Wine
                        uses its default, ~/.wine
     [game] display     the X display the game runs on. "auto" means
                        whatever $DISPLAY is

   Leave the rest alone. There's no shim path in the example on purpose:
   ./stasis play always uses the bundled shim/dinput8.dll and pins its
   checksum, in a copy of your config it keeps with that start's logs. Edit
   your own stasis.toml, never that copy.

4. Know what Stasis writes into your game folder. The top of the example
   config lists it too:
   - dinput8.dll, the shim, on every start. If another mod's dinput8.dll is
     there, Stasis moves it to dinput8.dll.stasis-backup-<id> first and tells
     you. It never deletes it. To put your mod back, rename the backup to
     dinput8.dll.
   - swkotor.ini, sound and display settings for the run.
   - Saves, but only if you set pin. A pin folder's files replace the game's
     own on every start, Saves included, so your current saves get
     overwritten. Back them up before you use pin.
   - Speedrun saves, only when you run ./stasis saves install (see SPEEDRUN
     SAVES AND RUNS). They go into Saves as new folders, and the list of what
     went in is stasis-speedrun-saves.json, next to Saves.
   Don't copy shim/dinput8.dll into the game folder yourself. The launcher
   does it every start, and it stays there afterwards.

5. Check it:

     ./stasis check stasis.toml

   Nothing gets started or changed. You get one line per test: "ok", "note"
   (a warning) or "fix". It checks:
   - the runtime, and the bundled shim against VERSION
   - that the config is valid
   - Wine, setarch, and that the Wine prefix exists
   - that swkotor.exe is KotOR 1 GOG 1.03. It hashes the exe's code, the same
     test the shim does
   - no KPM, and whether another mod's dinput8.dll will get moved
   - that the display answers
   - that the recordings folder is writable and has room
   - that ports 47610 and 47611 are free
   - the window toolkit
   It ends with "Ready to play." or how many things to fix. Every "fix" line
   says what to do.


STARTING IT
-----------

    ./stasis play stasis.toml
    ./stasis --help
    ./stasis --version

What play does, in order:

1. The same checks as ./stasis check. If any fail, it stops and lists them.
2. If something is still running in the same Wine prefix (the last game
   still closing, say), it says "waiting for the last game ... to exit" and
   waits up to 60 seconds. After that it gives up with "Another program is
   still running in the Wine prefix". Close that and start again.
3. It makes a folder for this start under ~/.local/state/stasis/runs/. That's
   where the config copy and the logs go. It keeps the last 10.
4. It puts the DLL in place and starts swkotor.exe under Wine. The terminal
   says "Starting the game with ... This takes 1-2 minutes." Believe it.
5. The game boots 60 frames and stops at the main menu. The terminal says
   "At the main menu. Load your save."
6. "Ready on 127.0.0.1:47610 ..." means it's serving, and the window opens
   over the game, paused. Until the game answers, the bar says "Waiting for
   the game".

Getting to your save: Stasis drives the game, so you get there with inputs
like anything else. Use mouse clicks in the piano roll (More > Editing
controls, then Mouse edit), or a little score of m@X,Y lmb lines loaded with
Load score here (see SCORES). Starting a recording straight from a save isn't
in this version. Every take starts at the main menu, and the load is part of
it. More > Speedruns lists the speedrun saves and tells you which row to pick
(see SPEEDRUN SAVES AND RUNS).

--load-game6 does the old thing: 16 fixed clicks that load the Game6
reference save. The clicks are blind. They land on whatever save sits in
that spot, so only use it if you have that exact save.

Options for play go after the config file:

    --load-game6        load the Game6 reference save (see above)
    --interface saved|visible|hidden
                        start the window shown, hidden, or the way you left
                        it
    --ui-config FILE    your own layout file. The default is the bundled
                        lib/stasis_tools/compact-ui.toml
    --no-ui             start the game without the window. It prints the
                        command to attach one later
    --close-with-ui     closing the window closes the game too
    --port N            command port, default 47610 (N+1 is the preview)

STASIS_MIN_FREE_GB and STASIS_WARN_FREE_GB change the disk limits (2 and 10
GB).

To quit, press Ctrl+C once in the terminal. It closes at the next safe point
so the recording ends clean. Press Ctrl+C twice and it stops immediately.
Closing just the window leaves the game running, unless you started with
--close-with-ui. The terminal prints the command to bring the window back.


THE INTERFACE
-------------

Ctrl+Shift+F12 shows and hides it. Hiding is cosmetic. The game doesn't move
and nothing gets saved or lost. More > Hide Stasis does the same thing.

If your desktop has no system tray and you're on the see-through overlay, hide
refuses and tells you to use the minimal bar instead. That's on purpose. A
hidden window with no tray icon has no way back.

The bar along the bottom has ▶ Play, ◀ Back, Advance ▶ and More next to the
seek bar, plus "Frame N · Saved M": the frame you're on and the last frame
that's on disk. Open the full panel and you also get Edit inputs and Save
inputs. Almost everything else is under More.

The first time, a tip says "Ctrl+Shift+F12 shows or hides Stasis. F1 lists
all keys." The ✕ gets rid of it for good.

Click the Stasis window to give it keyboard focus. Window-only keys like F1
and the live score keys need that. During playback it doesn't take focus.

Press F1 for the full key list, or use More > Keyboard shortcuts...

The keys you'll use most:

    Ctrl+Shift+F12    show / hide Stasis
    Ctrl+F7           play / pause
    Ctrl+F8           frame advance (hold it to repeat)
    Ctrl+F9           frame back (rewind one frame)
    Ctrl+F6           open / close the input editor
    Ctrl+Alt+Return   apply the selected draft
    Ctrl+Alt+S        save your input edits
    Ctrl+Z            undo an input edit
    Ctrl+Shift+Z      redo (Ctrl+Y works too)
    Ctrl+G            go to frame
    Ctrl+T            turbo on / off
    Ctrl+,            settings
    F1                the shortcut list

The Ctrl+Alt ones and Ctrl+Shift+F12 work whether the game window or the
Stasis window has focus. In the shortcut list those are marked "Game focus
too". Everything else needs the Stasis window focused.

Play
  Runs frames forward with whatever the piano roll says for each one. Empty
  frames mean nothing held. Play sends frames in batches that grow while
  things run smoothly. Pause lands at the end of the batch already in flight,
  so it can stop some frames after you hit it. Settings > Playback caps the
  batch size. If you need one exact frame, pause early and use Frame advance.

Advance ▶ (frame advance)
  Plays exactly one frame with that frame's input, then stays paused. Hold
  the key, and after a short delay it keeps stepping.

Turbo (Ctrl+T)
  Plays and replays in bigger batches with fewer screen updates. It's the
  only fast mode there is. Pause takes longer to land with it on.

◀ Back, the seek bar and jumping around
  ◀ Back restores the previous frame. The seek bar spans the whole
  recording. Drag it back to restore an earlier frame. Drag it forward and
  Stasis replays the recorded inputs up to there. The counter reads like
  "Frame 420 of 900 · 3 rerecords".
  - Ctrl+G goes to a frame by number.
  - Ctrl+. stops a replay at the next frame.
  - Alt+Left and Alt+Right move back and forward through where you've been.
  - Ctrl+R returns to the frame you were at before your last jump back.
  Every restore is checked. If a check fails, playback stops and you get an
  error instead of a game that's quietly wrong. A rewind can take tens of
  seconds. That's normal right now.

Takes
  A rewind starts a new take. Rewind live-1.holo to frame 420 and Stasis
  closes live-1.holo as it was and carries on in live-1@b1@420.holo next to
  it: session live-1, branch 1, frame 420. Nothing gets overwritten, so
  rewinding is always safe. It does make a lot of folders. The Takes page
  (More > Takes) lists them with their rerecord counts, and you can open one
  for playback or give it a note.

  Branch here, Open and Mark are greyed out in a live session. They belong
  to the viewer for old recordings.

  The Recording row under More has Browse..., Load recording and Save
  recording. Save recording makes sure the take is on disk. Load recording
  swaps in another one. Any unsaved input edits get parked in a recovery
  file first, so you don't lose them.

Piano roll
  Edit inputs (or Ctrl+F6) opens it. Frames go left to right, and each
  row is a key.
  - Arrow keys pick a cell. Space, Enter or double-click toggles it.
  - Your edits are drafts. They show up in the roll, but the game doesn't see
    them until you step or play over that frame, or hit Apply selected
    (Ctrl+Alt+Return).
  - Ctrl+Z undoes, Ctrl+Shift+Z or Ctrl+Y redoes. More > Edits lists them.
  - Save inputs (the button, Ctrl+Alt+S, or Ctrl+S inside the roll) writes
    your drafts to disk. That saves your inputs, not the recording.
  - The toolbar with Apply selected, Add lane and Mouse edit is hidden until
    you tick More > Editing controls. Add lane takes a key name (lshift, f4,
    space...) and gives it a row.
  - Follow playhead keeps the current frame in view.

Range edits (piano roll focused, on the selected frames)
    Ctrl+C / Ctrl+X / Ctrl+V    copy / cut / paste
    Ctrl+Shift+V                paste and push later inputs right
    Insert                      paste mode: insert or overwrite
    Delete                      release the selected keys
    Ctrl+Insert                 insert blank frames
    Ctrl+Delete                 delete frames, later inputs move left
    Ctrl+D                      clone the frames right after themselves
    Ctrl+H                      hold the selected keys on every frame
    Ctrl+Shift+H                fill a press / release pattern
    Ctrl+L                      lock keys so edits leave them alone
    M                           add or edit a bookmark
    Q                           read-only: Step and Play only replay
  Shift+PgUp and Shift+PgDn jump between bookmarks. The More > Timeline menu
  has all of these too.

Other things under More
  - Show keyboard details: the keyboard and mouse state the game actually got
    on this frame. Use it when you think an input didn't land.
  - Console: type commands. play, pause, stop, step, rewind, seek 420, save,
    or input 420 {"held":["w"]} to set frame 420's keys.
  - Switch layout, the Layout row (Load layout, Save layout) and Reload layout
    file. Saved layouts live in ~/.config/kptas/layouts.
  - Show minimal bar and Show compact timeline, for when you want it out of
    the way.
  - Memory... (Ctrl+Alt+M): watch addresses and search memory. It's
    read-only. Values come from the recorded frames, and freezing a value is
    off in TAS runs.
  - Score text: the score as text, in the editor.
  - TAS debugger...: what inputs the game confirmed, and run speed.
  - Speedruns... (Ctrl+Alt+J): the speedrun list. See SPEEDRUN SAVES AND
    RUNS.
  - Live score, Input trainer, Sequence file guide...: see below.
  - Settings... and Keyboard shortcuts...: see KEYMAP AND SETTINGS.
  - Reconnect: only lights up when Stasis lost track of a command (see
    TROUBLESHOOTING).

Engage TAS (Ctrl+Shift+F11)
  Meant to hand the game to your real keyboard and back. It's off in 0.0.1b,
  and the menu item stays greyed out. That's expected.


WHAT "FRAME PRECISION" MEANS
----------------------------

One frame is one pass of the game's main loop. That's one game update, and
usually one picture on screen. 60 of them are one second of game time.

Frame precision means Stasis steps, records and restores at that grain. It
isn't half-frames and it isn't "about 16 ms". The counter is the absolute frame
number since the game started. Frame 0 is the stop before any input, and the
first input lands on frame 1.

There's a finer mode, [clock] precision = "tick" (1000 per second), and the
interface says "Tick" everywhere when it's on. The live score is built for
frames. Leave it on "frame".

On saves: an in-game save takes zero frames, but the game still sees time
pass. By default a save is worth 900 ms of game clock. That's my estimate,
not a measurement, and it's the save teleport's fuel. save_ms=N in a score
header or @save ms=N for one save changes it. Per KNOWN ISSUES, save tricks
aren't reliable in 0.0.1b.


SCORES
------

A score is a text file of inputs. Each line is a note: how many frames to hold,
then everything held during those frames. Notes are stateless. You list
everything held, not what changed. A dot means nothing is held.

    #!score v1 game=k1 hz=60
    # walk forward, run a bit, stop, quicksave
    @mark start
    30  w
    10  w lshift
    4   .
    1   f4
    20  .

That's 30 frames of W, 10 frames of W plus Shift, 4 frames of nothing, one
frame of F4 (quicksave), then 20 frames of nothing.

What goes on a line:

    w  f4  space  lshift  esc  enter   a key, held every frame of the note
    lalt+f4                            a chord, pressed in that order
    lmb  rmb  mmb                      mouse buttons, held
    m@400,330                          cursor held at 400,330
    md5,-2                             cursor moved by 5,-2 every frame
    wh+1  wh-1                         wheel, first frame only
    !altf4  !altf4@save  !close        window events, first frame only
    "Revan"                            types text, one char per COUNT frames
    $qsave                             an alias

Key names are lowercase. shift, ctrl, alt, escape and return are accepted and
turned into lshift, lctrl, lalt, esc and enter.

Directives, one per line:

    @mark NAME          names this frame
    @segment NAME       starts a section (hz=, pace=gate or pace=realtime)
    @sync EVENT         waits for the game instead of counting frames:
                        main_menu, area.ready, load.done, save.begin,
                        save.done, dialog.open, dialog.close, gui.open PANEL
    @include FILE       pulls in another score
    @alias NAME = KEY   then use it as $NAME
    @repeat N ... @end  repeats the lines between
    @save ms=N          save length for the next save only
    @expect OUTCOME     exit (game must quit), alive, or exit|alive
    @relaunch           starts the game again (if=exit: only if it exited)
    @assert F OP V      checks a recorded value, like @assert pc.x >= 12.5

# starts a comment. The KotOR 1 key bindings ship as aliases in
the bundled k1-keymap.score: @include k1-keymap.score, then $qsave, $pause
and so on. Stasis finds that file on its own. If you rebound keys in the
game, it's wrong for you.

Checking a score:

    ./stasis score check my.score

It prints an ok line per file, or the error with file, line and column and a
guess at what you meant:

    ok my.score: 6 items, 65 note frames, 1 files, sha256 850802cb6e432e27
    telos: bad.score:2:3: unknown key 'wq' (did you mean 'w'?)

Live score: playing a score in the session you have open
  More > Live score:
    Load score here...   Ctrl+Alt+O   start a score at the current frame
    Load score at...                  go back to a frame you pick, start there
    Pause score          Ctrl+Alt+P   pause; the same key resumes
    Unload score         Ctrl+Alt+U   stop it here, the game stays put
  These keys need the Stasis window focused. While a score is loaded, the
  play / pause key (Ctrl+F7) pauses and resumes the score too, and the
  Play button reads "Stop run" or "Resume score".

  - Loading a new score replaces the one that's loaded.
  - When a score is stopped or unloaded early, the inputs it didn't play yet
    are kept as a draft .score in ~/.local/state/kptas/score-drafts/, named
    like mything-rest-from-frame-900-....score. Load that wherever you are.
  - Includes work like the checker: next to the score, then the bundled
    keymap.
  - Notes, @mark, @repeat, @alias, text and @include are fine.
  - What it refuses, with a message that says why:
    - a header that sets save_ms, tier, render or keymap, or a start= other
      than boot
    - hz other than 60, or a @segment that changes timing
    - @sync, @expect, @relaunch, @save, @assert
    - window events like !altf4
    - lines with 0 frames

  A full score with @sync and friends can be written and checked in 0.0.1b,
  but there's no command that plays it start to end on its own.


SPEEDRUN SAVES AND RUNS
-----------------------

The community speedrun save packs on speedrun.com are route checkpoints:
ChaosDrifter's Any% Unrestricted 10.1 saves and Lane's Any% 9.4 saves. They
hold no inputs. What they give you is a known place to start from.

Putting them in your game:

    ./stasis saves install --game /path/to/game
    ./stasis saves list --game /path/to/game
    ./stasis saves remove --game /path/to/game

- Install checks each pack zip against the checksum we recorded when we
  first downloaded it. If the zip is missing or different, it downloads it
  again from speedrun.com.
- Both packs use the same folder names, so each slot gets a new number.
  ChaosDrifter's slots become 000100 to 000120, Lane's 000202 to 000222.
- The name in the load list gets the pack in front, like "Chaos 10.1:
  Routine" or "Lane 9.4: ROUTINES". Only that name changes. Add --keep-names
  to skip it.
- It never touches a save it didn't put there. Run it again and it skips
  what's already in.
- Remove deletes only its own folders, and only if they're unchanged. A slot
  you saved over stays.
- If your config sets pin, every start copies the pin's Saves over the
  game's, and saves the pin doesn't have are removed. Install into the pin
  folder too, or they're gone at the next start.

The game should list saves by folder name. That comes from reading the
game's code, and I haven't checked it on screen yet. If it holds, your own
saves numbered 2 to 99 stay at the top, and --load-game6 still finds Game6 in
row 1. Check that row 1 is Game6 before you use --load-game6.

The speedrun list: More > Speedruns... (Ctrl+Alt+J), or ./stasis runs list.
It lives in ~/.config/kptas/speedruns.toml.
- Import runs adds the two save packs and the runs we've pulled inputs from
  videos so far (R4NG3 and Otterito), with runner, date and run ID.
- Each run starts with "New (start recording)", then its checkpoint saves,
  its scores and any takes you attached.
- New doesn't load the save for you yet. It shows the steps: start a
  session, Load Game, then the row to pick. For Game6 it gives you the
  --load-game6 command instead.
- Add run, Rename and Remove edit the list only. Remove never deletes a save
  or a recording.

About the inputs. A score only means something from the exact spot it was
recorded from. The packs give you those spots for their own routes. The
video inputs come from other people's runs, and no save in the packs matches
where they start. So each video score is one section that still needs its
start point. None of them are replay-checked yet. The list shows each
score's status as it came in, and nothing gets marked as working until a
replay proves it.


INPUT TRAINER
-------------

More > Input trainer, or Ctrl+Shift+F9 with the Stasis window focused. It's a
combo-trial list. Each trial is a chain of inputs with timing windows. The
trainer watches every frame that actually runs (Step, Play, a loaded score,
replaying a recording) and tells you whether each input landed: Early by N,
Late by N or Perfect, in frames and in ms.

Keys (Stasis window focused, all changeable in Settings):

    Ctrl+Shift+F5    start / stop the trial
    Ctrl+Shift+F6    reset the trial
    Ctrl+Shift+F7    previous trial
    Ctrl+Shift+F8    next trial
    Ctrl+Shift+F9    show / hide the trainer

The Trials menu in the panel has Open trial files..., Make a trial from
frames... (pick a frame range and every frame with a new key press becomes a
step), Save this trial as a file..., Remove this trial from the list, Clear
stats for this trial, and Settings... Practice mode loops and starts over
after a miss. Strict gives you one try. Animations follow Settings > Reduce
motion.

A trial file:

    version = 1
    id = "walk-stop"
    title = "Walk, then stop 12 frames later"
    precision = "frame"
    start = "explicit-boundary"

    [[steps]]
    id = "go"
    label = "Press W"
    kind = "input"
    keys = ["w"]

    [[steps]]
    id = "stop"
    label = "Let go of W"
    kind = "input"
    match = "release"
    keys = ["w"]
    target = 12
    window_early = 1
    window_late = 2

Step fields:
- keys (and buttons for lmb / rmb / mmb). match is press, release or hold.
- target = N: N frames after the step before landed. window_early and
  window_late say how many frames off still count (2 each by default; set
  both to 0 for a one-frame window).
- Or the old way: window_min and window_max, counted from the step before.
- hold_min and hold_max for press steps: hold at least N frames, let go
  within N frames.
- chord = "contains" lets other keys be down too. The default, "exact",
  means only those keys.
- unrelated_inputs = "reject" at the top turns any extra key press into a
  miss.

The older More > Sequence file guide... still opens .sequence.toml files and
ticks steps off one by one. It doesn't know target, window_early,
window_late, hold_min or hold_max. The trainer loads old sequence files too.
While the guide has a file open, Play goes one frame per batch, so expect it
to be slower.


KEYMAP AND SETTINGS
-------------------

Ctrl+, or More > Settings... opens it. Tabs:

- Keys: every action with its category, current key and default. Search by
  action or key. Click a key and press the new one. Esc cancels, Backspace
  removes it. Clear key, Reset key and Reset all keys do what they say.
  Import... and Export... move your keys around as a .toml file.
- Display: layout, game picture, detail, timeline edge, follow playhead,
  panels, panel opacity, window position, colors, and Reduce motion.
- Playback: the largest Play batch in frames and the longest in time. Make
  them smaller if Pause overshoots too far for you.
- Shortcut list: the cheat sheet. Every action and what it's bound to. F1 or
  More > Keyboard shortcuts... opens Settings right on it.

Your keys are saved in ~/.config/kptas/keymap.toml. If you edit that file by
hand, Ctrl+Alt+R reloads it without a restart.

Keys marked "Game focus too" also work while the game has focus. Everything
else only works with the Stasis window focused. If a key does nothing, your
desktop may be grabbing it first. Bind that action to something else.

The window layout itself (sizes, panels, which layout) comes from
lib/stasis_tools/compact-ui.toml. To use your own, copy that file somewhere
of yours, edit the copy, and start with --ui-config:

    cp lib/stasis_tools/compact-ui.toml ~/stasis/my-ui.toml
    ./stasis play stasis.toml --ui-config ~/stasis/my-ui.toml

Don't edit compact-ui.toml in place. It's listed in MANIFEST.sha256, and the
next release overwrites it anyway.


RECORDINGS
----------

Recordings go in [run] out_dir from stasis.toml. The example sets
~/stasis/recordings. Delete that line and they go to ~/.local/share/kptas/runs
instead.

- Each session is live-1.holo, live-2.holo and so on. A .holo is a folder,
  not a file.
- Each rewind adds a take next to it, named session@branch@frame:
  live-1@b1@420.holo, live-1@b2@300.holo and so on.
- A take needs its parent for the shared history. Keep them in the same
  folder. Don't move just one.
- Each take has a storage budget, 18 GB by default. Past that it stops with
  "store budget of N bytes: ... did not make room". Real sessions are far
  smaller (see KNOWN ISSUES), but watch free space anyway.
- When a save notice reports a frame, that's the last frame actually on disk.
  The counter can be ahead of it. Use Save recording before you do anything
  you might regret.

Logs live somewhere else. ~/.local/state/stasis/runs/<time>-<config name>/
has the config copy, ui.log and kptas_shim.log for that start. The last 10
starts are kept.


TROUBLESHOOTING
---------------

Start with ./stasis check stasis.toml. Most setup problems show up there with
a "fix" line that says what to do.

"Found KPM files: ... Stasis does not run with KPM installed."
  KPM is in your game folder. The message lists every file it found, then the
  exact steps for your folder. Do those steps, and only those. Don't go
  deleting binkw32.dll on a hunch, because in a clean install that's the
  game's real video DLL.
  - Only when the message names both binkw32.dll (as KPM's KProxy) and
    binkw32Hooked.dll does it tell you to delete binkw32.dll and rename
    binkw32Hooked.dll back to binkw32.dll.
  - KotorPatcher.dll, patch_config.toml and the patches folder just get
    removed.
  Stasis never moves or deletes any of it for you. If you're unsure, your
  store's verify-files option puts the real binkw32.dll back.

"The shim refused to run. KPM was loaded from outside the game folder ..."
"The game exited with code 75 ..."
  Same thing, caught later: KPM got in through another folder, a Wine DLL
  override, or a game started from somewhere else.

"The shim refused to run. Something other than Stasis patched swkotor.exe at
start."
  Another patch tool got to the exe first. Remove it and relaunch.

"... is not KotOR 1 GOG 1.03"
  Wrong exe. Stasis only runs that exact one.

"the shim did not connect within 120 s"
  The game started but the DLL never called home. Look at ui.log and
  kptas_shim.log in that start's run folder (see RECORDINGS).

"Session error" in the top right corner
  Hover it or click it for the real message.
  - "Stasis lost track of the last command, so playback stopped. Choose
    More > Reconnect to check the game and continue." Do exactly that.
  - "Stasis cannot tell whether the game is still running, so playback
    stopped. Choose More > Reconnect." Same.
  - "The game stopped responding, so Stasis stopped playback. Restart Stasis
    to continue." The game is gone. See the next entry.

The game window is gone, or it's black and nothing answers
  The display the game was on died, or the game crashed with it. Stasis can't
  bring a dead game back. Your recording is on disk up to the last saved
  frame. Ctrl+C the launcher, start ./stasis play again, then Load recording.

"tas-live: the disk is full ..."
  The recording couldn't be written, so Stasis stopped. Free some space, or
  point [run] out_dir at a bigger disk, then start again. Anything after the
  last saved frame is gone.

It's slow
  About 25 frames per second is normal in 0.0.1b (KNOWN ISSUES). Slower than
  that usually means something else is hammering the same disk: a backup, a
  file indexer, a torrent, a second Stasis. Stop the other thing, or put
  recordings on another disk. A rewind taking tens of seconds is normal too.

"Ports 47610, 47611 are in use"
  Another Stasis is running. Close it, or pass --port with a free port.

"Another program is still running in the Wine prefix ..."
  Something is still alive in that prefix 60 seconds later. Close it, or kill
  the PIDs it names, and start again.

"PySide6 not found."
  Install python3-pyside6 with your package manager, or PySide6 with pip.

A key does nothing
  - Check where focus is. Most keys need the Stasis window. F1 shows which
    ones also work from the game.
  - "Tool shortcut waits for an acknowledged owner state": it's mid-command.
    Wait for it.

The interface is gone
  Ctrl+Shift+F12.

"Inputs not saved: another TAS window is saving ..."
  You have two windows on the same session. Close one.


REPORTING A PROBLEM
-------------------

Steps from start to finish make a problem fixable. Without them it usually
isn't.

Include:
- what you did, the exact keys or buttons, in order
- what you expected, and what actually happened
- the frame number on screen when it went wrong
- the score or trial file, if one was involved
- the terminal output from ./stasis
- the output of ./stasis --version, your distro, and wine --version
- your run folder from ~/.local/state/stasis/runs/ for that start. It has
  ui.log and kptas_shim.log, and the terminal prints its path when play
  starts

Leave the .holo folder out unless someone asks for it. It can get big.


KOLTO LABS
----------

Website: https://koltolabs.bocloud.workers.dev/
GitHub:  https://github.com/kolto-labs

Stasis is not open source, so there's no Stasis repository to look for.


Copyright (c) 2026 Kolto Labs. All rights reserved.

Stasis is a fan-made tool. It is not affiliated with or endorsed by
Lucasfilm, Disney, BioWare, Aspyr, or EA. Star Wars and Knights of the Old
Republic are trademarks of their respective owners.

Stasis is not open source. Do not redistribute this archive without
permission from Kolto Labs.
