# Channels and Notes

A "song" is, at minimum, a track and some notes. At its simplest, a track consists of a header with `#channel=N` (where N is a number from 1 to 16), and then a list of notes to play on the next line(s). These notes will be treated as eighth notes.

```
#channel=1
CDEFGABC
```

# Note Duration

- You can use dashes (`-`) to extend a note by one eighth (of a measure) at a time.
- You can place a note in parentheses (`()`) to halve its duration.
- Use spaces (` `) or bars (`|`) to help organize notes. These don't impact timing.

The following example plays up and down the scale starting with half notes, then with quarter, eighth, 16th, and 32nd notes.

```
#channel=1
C---D---E-F-G-A-BCDEFGAB
(CBAGFEDC (DEFG) A-B-) C
```

# Rests

You can use dots (`.`) to indicate rests, i.e. when a note should not be played.

```
#channel=1
C-D.E-F.G-A.B-C.
```

# Specifying Octave

By default, the current note is higher- or lower-pitched based on proximity to the previous note.

- You can follow a note with up/down symbols (`^` or `v`) to specify that the note should be higher or lower, respectively.
- You can follow a note with a single-digit number to specify absolute octave. Note that octaves start with C, not A!

The first note, if not specified, is always on the octave that makes it closest to C5.

```
#channel=1
G - G^- C - D - C - G - Gv- . .
G - G^- Cv- D - C - G^- Gv- . .
C2- C3- C4- C5- A5- A4- A3- A2-
```

# Sharps and Flats

- You can use pound signs (`#`) to raise a note by one semitone.
- You can use lowercase b (`b`) to lower a note by one semitone.

These need to go before the octave number.

```
#channel=1
C - C#- D - Eb- E - F - F#- G - G#- A - Bb- B - C - . . . . . .
Bb2-Bb3-Bb4-Bb5-
```

# Chords

Chords can be expressed using chord notation surrounded by colons (`:`). For example, you can write `:Cmaj7:` or `:Abmin7:`. Writing just the note itself implies a major chord (like `:C:`).

Any accidentals must go inside the expression, but the octave must be specified outside. So to play C#maj with C#6 as the root, you'd write it as `:C#:6`.

If the chord you want isn't working, you can also use slashes (`/`) to separate notes that should be played at the same time.

```
#channel=1
(:Cmaj:4-.:Cmaj:4-.:Cmaj:5.)
(:Gmaj:4-.:Gmaj:4-.:Gmaj:5.)
(:Bbmaj:4-.:Bbmaj:4-.:Bbmaj:5.)
(:Fmaj:4-.:Fmaj:4-.:Fmaj:5.)
(:Emin7:4-.:Emin7:4-.:Emin7:5.)
(:Amin7:4-.:Amin7:4-.:Amin7:5.)
G4/B/D/F/A---
```
