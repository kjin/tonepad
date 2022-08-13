# Tonepad + ASC

ASC is a text-based file format introduced in this codebase that holds music composition information. Tonepad is a player that can load such files and send musical information to MIDI devices, or play it back locally via soundfonts. This makes it compatible with any MIDI-based DAW or hardware.

## UI

The Tonepad UI features an editor, playback controls, and more. At the moment, it's best viewed at 1024x768 resolution or higher, although components can be resized to fit as needed.

The Tonepad UI contains the following components:

- **File Loader**: Load or save a file. The given string is the "name" of the file.
- **Playback Controls**: Start, stop, or seek the current song.
- **Editor Settings**: Set settings for the editor. The following options are available:
  - _MIDI Typing_: Directly input notes from the currently enabled MIDI input.
  - _Optimal Insert_: When inputting notes, try to write the new note as concisely as possible. Without this, every note's absolute form will be inserted (e.g. `C5E5G5`), but with it, the octave number is dropped if it can be inferred (e.g. `C5EG`).
- **Device Selector**: Select a MIDI input or output device. Using the same device for both is currently not allowed. Note that even when there are no MIDI output devices, a `[Built-in Soundfont]` option is available to play songs using the browser-based soundfont player. (Note that this requires an internet connection.)
- **Channels**: Sample, solo, or mute channels. When the channel number is held, `C5` will be played for that channel. The last-clicked channel number will also receive inputs from the MIDI input device if one is selected.
  - Solo (yellow star icon): If at least one channel is solo'd, only solo'd channels will play notes. Shift+click a channel's solo button to only solo that channel.
  - Mute (red mute icon): Muted channels will not play notes.
- **Instrument Selector**: Select instruments that play when the output device is set to `[Built-in Soundfont]`. These are graciously pulled from the [midi-js-soundfonts](https://github.com/gleitz/midi-js-soundfonts) repo. Note that `-1`/`+1` buttons transpose the instrument in octaves.
- **Console**: Various messages from the system, including some ASC file parsing errors.

## ASC Syntax

A single ASC file represents a **Song**. A **Song** is made of one or more **Tracks** that play simultaneously. **Tracks** are made up of individual notes, rests, and **Patterns** -- which are composable sequences of other notes, rests, and patterns.

ASC is an entirely text-based format, divided into **Blocks**. There are **Header Blocks** and **Note Blocks**; **Header Blocks** are single lines prefixed with a **Header Character**, while **Note Blocks** are multi-line sequences delimited either by header blocks or a double-newline (`\n\n`). Header blocks define song, track, or pattern metadata, while note blocks describe a track or a pattern.

A `%` denotes the beginning of a **Comment**, and affects the remainder of the line.

### Header Blocks

Headers are prefixed with one of `!`, `@`, or `#`, followed by a number of `k=v` pairs:

- `!` denotes a song. The note block immediately succeeding it will be a _track_ (however, the use of "naked tracks" -- tracks that do not follow `#`-prefixed header blocks -- are discouraged).
  - `bpm=(number)`: Set the song's tempo. This is a value greater than 0.
  - `swing=(number)`: Add swing to the song. This is a value from 0 to 1.
  - `transpose=(number)`: Transpose all notes in the song. This is an integer value.
- `@` denotes a pattern. The note block immediately succeeding it will be a _pattern_, referrable with an ID denoted by the `id` key.
  - `id=(string)`: The pattern's name. This is how the pattern will be referenced.
    - If multiple patterns have the same ID, they will be layered on top of each other. Each "sub-pattern" maintains its separate remaining properties described below.
  - `channelIndex=(number)`: Set which channel in a track the notes in the pattern will play. Note that this isn't the channel number itself, but the index when multiple channels are specified for a track. This is an integer value at least 0.
  - `transpose=(number)`: Transpose all notes in the pattern (in addition to the global transpose). This is an integer value.
  - `velocity=(number)`: Set the velocity (~volume) for all notes in the pattern. This is an integer value from 0 to 127.
- `#` denotes a track. The note block immediately succeeding it will be a _track_.
  - `channel=(number)`: The channel that notes in the track will play on. This is an integer value at least 1.
  - `channels=[(number), ..., (number)]`: Additional channels that are specified for the track. These channels are only used to play notes specified in patterns with a specific `channelIndex`. These are effectively 1-indexed because the first channel (index 0) is always specified by `channel`. Each element is an integer value at least 1.
  - `transpose=(number)`: Transpose all notes in the track (in addition to the global transpose and per-pattern transpose). This is an integer value.
  - `velocity=(number)`: Set the velocity (~volume) for all notes in the track. This is an integer value from 0 to 127.

### Note Blocks

Note blocks can be sequences of notes, chords, rests, or patterns (collectively just called **Units**). By default, each unit immediately succeeds the previous unit temporally; in other words, they form a timeline. The following syntax is supported:

- Notes
  - `A`-`G` represents a note A thru G respectively. The note played is at the octave _closest_ to the previous note, after sharps and flats are considered. The first note is always the one closest to `C5`.
  - `#`/`b` raises/lowers the previous note by one semitone respectively. These may not be repeated.
  - `^`/`v` _forces_ the current note to be above/below the previous note, respectively, rather than just being at the closest octave.
  - `0`-`9`, when following a note, determines the absolute octave number. This cannot be specified at the same time as `^`/`v` for obvious reasons
- Chords
  - The structure of a chord, which is placed inside of these brackets, is: `:<NOTE><CHORD>:<OCTAVE>`.
  - `:`/`:` denote the start and end of a chord, respectively.
  - A `<NOTE>` is any combination of base note (with optional sharp or flat, but no octave information).
  - A `<CHORD>` can be one of the following: `maj`, `min`, `sus2`, `sus4`, `7`, `maj7`, `min7`.
  - **TBD** Additional chords.
  - **TBD** A notation to denote inversion.
  - An `<OCTAVE>` is one of the octave markers. It is placed on the outside to avoid a confusion with the `7` chord.
- Other Units
  - `0`-`9`, when not following a note, are _placeholder_ units, which will be replaced by real notes in pattern references. Due to the possible ambiguity with octave number, it is not recommended that notes and placeholder units be mixed.
  - `.` represents a single rest.
  - `-` represents an extension of the previous unit.
  - `*` represents a repetition of the previous note or chord.
- Time changes
  - `(`/`)` decreases/increases the duration of each note or rest by a magnitude of 2.
  - `/` makes the next unit play at the same time (precisely, start position) as the previous one.
- Time directives
  - Time directives denote timing points in the song. If the number of beats that have elapsed doesn't match the number contained in the time directive, it will be adjusted to fit (longer sequences are truncated, while shorter sequences are filled according to the presence of `@`). The structure of a time directive, which is placed inside of the brackets described below, is `{<MODS><NUM_BEATS>}`.
  - `{`/`}` denote the start and end of a time directive, respectively.
  - `<MODS>` can be one or more of the following, in order:
    - `!`: Make it an error if the number of beats that have elapsed doesn't match the number contained in the time directive.
    - `@`: Repeat all notes since the previous time directive until this time directive. Without this, the remaining space is filled with rests instead.
    - `+`/`-`: Make this time directive _relative_ to the previous time directive.
  - `<MODS>` can also be `x`, which means: Repeat all notes since the previous time directive, make this time directive _relative_ to the last time directive, and make `<NUM_BEATS>` represent the number of multiples of the time since the previous time directive, instead of the number of beats.
  - A `<NUM_BEATS>` is any number at least 0.
  - An empty `{}` denotes a time directive which has no effect up until its current point. It can be used as the "previous time directive" in a future time directive.
- Pattern references
  - Patterns are sequences of units that constitute a single unit, and can be defined independently or inline. The structure of a pattern, which is placed inside of the brackets described below, is `[@<PATTERN_NAME>|<SUBS>]` or `[<INLINE_PATTERN>|<SUBS>]`.
  - A `<PATTERN_NAME>` is the ID of a named pattern to insert here.
  - `<SUBS>` is one of the following:
    - A comma-delimited list of units that should be used to substitute `0`-`9` placeholder units in the pattern. Pattern and extend units are not allowed here.
    - A `^` or `v`, followed by exactly 12 characters ranging from `0` to `9`, as well as `A`, `B`, and `.`, indicates a harmonization of the notes in the pattern. The twelve characters denote the number of semitones to move a note in the chromatic scale, where the first character denotes `C` and so on. `A` and `B` here indicate 10 or 11 notes respectively, and `.` indicates to omit a note. Whether to move up or down is represented by `^` or `v`, respectively.
      - This can be followed by another harmonization (which stacks on the previous one) or subs -- all delimited with `|`.
    - A deconstructed chord, which is a chord (e.g. `:Cmaj7:`) or a group of simultaneous notes (e.g. `C/E/G/Bb`) followed by a list of numbers delimited by `{`/`}`. Each number represents a chord note index; for example, `:Cmaj7:{0,1,2,3}` will become `C`, `E`, `G`, and `B`. Each note then substitutes the first available placeholder unit. This allows for arpeggios.
      - If the chord note index is out of bounds, then it will behave as an index as if the chord continued to wrap with the same notes on both ends of the pitch spectrum. For example, `:Cmaj7:{-1,7}` is converted to `B` (shifted down one octave) and `C` (shifted up two octaves) respectively.
      - If a placeholder is specified, then the chord that is written as the substituted for this placeholder will be destructured. Note that behavior is undefined when a note or rest is substituted.
        - **TBD** This currently does not work for simultaneous notes.
  - An `<INLINE_PATTERN>` is a note block itself, and is parsed as a pattern. Note that because `|` is ambiguous, it cannot be used in inline patterns.
- **TBD** Effects
  - This whole thing is TBD. Some ideas:
    - Tremolo (`~`)
    - Legato (`_`)
    - Strum (?)
