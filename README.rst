MIDIUtil2
=========

Background
----------

The library [MIDIUtil|https://pypi.python.org/pypi/MIDIUtil/] by Mark Wirt is a
very nice way of creating standard MIDI files in Python. It's lightweight,
simple to get up-and-running with and it has the advantage of having notes
specified as _start time_ + _duration_ rather than _start time_ + _end time_
which is usual for most MIDI software.

It is however missing a few features, most notably reading data from existing
MIDI files. I've chosen to fork it into a new library to avoid having to
sub-class it every time I need those extra features. Thus, MIDIUtil2 is born.

This fork is distributed under the same permissive Open Source license as
the original MIDIUtil library.

Installation
------------

Installation follows similarly to that of MIDIUtil:

.. code:: bash

    git clone git@github.com:michgz/MIDIUtil2.git
    # or
    git clone https://github.com/michgz/MIDIUtil2.git

depending on if you want to use SSH or HTTPS.

To use the library one can install it on one's system:

.. code:: bash

    python setup.py install

Documentation
-------------

Please refer to the 
[documentation of MIDIUtils|https://midiutil.readthedocs.io/en/1.2.1/]. All
functions of the original library are still supported by MIDIUtil2.

Additionally, three new functions are added:

Add Marker
----------

 addMarker(track, time, text="")

    Add a MIDI marker event to the MIDIFile object
    Parameters:	

        track – The track to which the notice is added. Note that in a format 1
                file this parameter is ignored and the marker is written to the
                system track
        time – The time (in beats) at which marker event is placed.
        text – The marker text [String]. May be left empty


Add End-Of-Track
----------------

 addEndOfTrack(track, time)

    Add a end-of-track event to the MIDIFile object. If omitted, the end of
    track is simply at the same time as the last event of the track. In some
    cases though it is useful to enforce a minimum track length, even if
    the final portion of it has no events.
    Parameters:	

        track – The track to which the notice is added. Note that in a format 1
                file this parameter is ignored and the same end-of-track time is
                assigned to all tracks, including the system track.
        time – The time (in beats) at which notice event is placed.

Read MIDI
---------

 readFile(fileHandle)

    Reads a MIDI file into a MIDIFile object. Currently only Type-1 MIDI files
    can be read.
    Parameters:

        fileHandle - a file handle that has been opened for binary read.


Quick Start
-----------

For quick start on creating and writing MIDI files, see
[MIDIUtil|https://github.com/MarkCWirt/MIDIUtil].

Reading a Type-1 MIDI file is as simple as calling `readFile()`. Tracks within
the MIDI file can then be iterated over or replaced, and the MIDI file can be
written back to disk as normal.

As a basic example, the following reads a file, transposes all notes in Track 2
up by 1 semitone, and then saves to a different file.

.. code:: python

    #!/usr/bin/env python

    from midiutil2 import MIDIFile, MIDITrack
    
    # Read from input file
    with open('INPUT.MID', 'rb') as f:
        MyMIDI = MIDIFile.readFile(f)

    NewTrack = MIDITrack(True, True)

    for Event in MyMIDI.tracks[2]:  # 0 is the system track, so this is the
                                    # second non-system track

        if Event.evtname == 'NoteOn':
            # Transpose all notes up 1 semitone
            NewTrack.addNoteByNumber(Event.channel, Event.pitch + 1, Event.tick,
                                Event.duration, Event.volume,
                                Event.annotation, Event.insertion_order)
        elif Event.evtname == 'NoteOff':
            # No action needed. All note off events are already accounted for
            # by NoteOn duration
            pass
        else:
            # Other events are used without change
            NewTrack.eventList.append(Event)

    # Replace the old track with new
    MyMIDI.tracks[2] = NewTrack

    # Write to output file
    with open('OUTPUT.MID', 'wb') as f:
        MyMIDI.writeFile(f)
