# Introduction

``bookification`` is a simply python script designed to combine ``.m4a`` audio
tracks into a ``.m4b`` audiobook that works well with Apple Books. The script
is designed as a post-processor for [abcde](https://abcde.einval.com/wiki/),
which together depend on several open sources tools to rip, combine, and tag
the audiobook.

# Setup

The process of ripping an audiobook you **own** on CD to a digital audiobook for
Apple Books requires several pieces of software that are all available from
[Homebrew](https://brew.sh/) or [CPAN](https://www.cpan.org/). First, install
Homebrew and then

    brew install abcde
    brew install cd-discid
    cpanm install MusicBrainz::DiscID
    brew install fdk-aac-encoder          
    brew install cdparanoia
    brew install flacc
    brew install cdrtools
    brew install atomicparsley

Create the configuration file ``~/.abcde.conf`` from the file ``dotabcde.conf``
in this git repo. This configuration file is tailored for ``bookification`` and
only has settings for producing ``.m4a`` files with the ACC encoder. The are
many more examples of abcde configuration files available on the Internet for
other uses.

Update the OUTPUTDIR path in your ``~/.abcde.conf`` file to point to the
location where you want the individual audio track files to be written. Once
all the tracks are ripped from CD, ``bookification`` will read the individual
tracks from this location and write a new output file.

# Rip all tracks

Use the ``abcde`` script to rip the individual audio tracks, collect album
artwork, download track names, and create a playlist. For the first disk,
just accept all the defaults. For each additional disk, when prompted,
choose to *append* to the playlist.

    abcde -W 1        # disk 1: accept all the default responses at prompts
    abcde -W 2        # disk 2: choose (a) for append to playlist

    Erase, Append to, or Keep the existing playlist file? [e/a/k] (e): a
    
    abcde -W 3        # disk 3: choose (a) for append to playlist
    ...

# Bind the audio tracks into a single audiobook file

Once all the tracks are ready, run ``bookification`` with three options:

    ``src``   - path to the album directory created by the ``abcde`` process
    ``dst``   - path to store the final audiobook file
    ``title`` - method for extracting the chapter titles from track titles

The first two options are self-explanatory, but the ``title`` option requires
some explanation. ``abcde``, by default, uses
[Musicbrainz](https://musicbrainz.org/) to determine the track titles. Most
audiobook CDs split one chapter across multiple tracks. For example, the
audiobook _Harry Potter and the Deathly Hallows_ read by Jim Dale comes on 17
discs. Musicbrainz labels the six tracks of Chapter 1 as:

    Chapter 1: “The Dark Lord Ascending”, Part 1
    Chapter 1: “The Dark Lord Ascending”, Part 2
    Chapter 1: “The Dark Lord Ascending”, Part 3
    Chapter 1: “The Dark Lord Ascending”, Part 4
    Chapter 1: “The Dark Lord Ascending”, Part 5
    Chapter 1: “The Dark Lord Ascending”, Part 6
    
``bookification`` must determine with tracks go together to form a single
chapter and the actual chapter title from these track names. The ``--title``
option is a regular expression with capture groups marking the parts of
the track title that will be joined together to determine the chapter title.
Tracks with the same title are combined into a single chapter.

The default value for ``--title`` is a regular expression for this example
that also removes the quote characters from the title:

    ^(Chapter [0-9]+: )“(.*)”, Part [0-9]+$

This regular expression captures the chapter number, colon, space, skips the
quote, and then captures everything up to but not including the final "quote
comma Part number" portion of the track title. A regular expression explorer
such as [pythex](https://pythex.org/) is a quick way to test out a regular
expression against track titles.

The ``abcde`` configuration file in this repo uses ``~/Audio`` as the output
directory for tracks. To bind _Harry Potter and the Deathly Hallows_ into a
audiobook file, we

* work in a separate working directory, ``work``
* write to the subdirectory ``audiobook`` in the working directory
* include the default ``--title`` option for illustration 

    terminal> cd ~/work/
    terminal> path/to/bookification \
    --src ~/Audio/m4a/J.K.\ Rowling\ read\ by\ Jim\ Dale/Harry\ Potter\ and\ the\ Deathly\ Hallows \
    --dst audiobook \
    --title='^(Chapter [0-9]+: )“(.*)”, Part [0-9]+$'

When finished, the subdirectory ``audiobook`` will contain temporary
copies of all the track files as well as the final ``.m4b`` file.  Move
the final ``.m4b`` file to another location and remove the ``audiobook``
subdirectory.


