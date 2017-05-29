SteamWorld Heist Savegame Editor
================================

This is a little utility to allow you to cheat at the excellent game
*SteamWorld Heist* by editing a savegame.  This is currently only tested
against savegames generated by the Linux version, but I would expect
it to work against savegames on other PC platforms as well.

The following values can be changed from the commandline:
 * Water (money)
 * Character XP (will be set to 19,999 for all unlocked characters)
 * Inventory size
 * Adding specific inventory items
 * Adding a ludicrously-huge chunk of powerful endgame items en masse

This utility is written in Python, using Python 3.  I didn't intentionally
use anything which would make it not work in Python 2, but it's totally
untested in Python 2.  It's command-line only; there's no GUI, nor do I
plan to write one for it.  I use it on Linux, but it should work in OSX's
Terminal without problems, or in a Windows commandline/powershell.

**Disclaimer:** This has been tested on a relatively small pool of
savegames that I'd collected once I realized I wanted to write this,
and I am positive that there are plenty of savegame possibilities that
this utility won't be able to read.  The utility uses Python's `assert`
statement pretty frequently while loading, and does a sanity check to
make sure that it can correctly write out a 100%-accurate replica of
the savegame before actually making any changes, so the worst-case
scenario is that the utility Just Doesn't Do Anything, but as always,
keep in mind that this utility could generate a savegame that will
break *SteamWorld Heist* in unexpected ways.  Buyer beware!  Make backups
of your savegames!

This utility does *not* parse the entire savegame; it only goes up to
about the place in the file that I was interested in editing, so there's
about half the savegame beyond that point which the utility just treats
as raw data.

This is released under the New/Modified BSD License.  See `COPYING.txt`.

Usage
-----

```
usage: swh_savegame.py [-h] [-l | -c | -o OUTPUT] [-e] [-w WATER] [-s SIZE]
                       [-i] [-n] [-a item[,item,...]] [-v]
                       FILENAME

View/Edit SteamWorld Heist Savegames

positional arguments:
  FILENAME              Input filename

optional arguments:
  -h, --help            show this help message and exit
  -l, --list            Just show information about the savegame
  -c, --check           Just check that we can load the file and can produce a
                        100%-accurate replica
  -o OUTPUT, --output OUTPUT
                        Output filename (required if making changes)
  -e, --experience      Set all unlocked character XP to 19999
  -w WATER, --water WATER
                        Set Water value (money)
  -s SIZE, --size SIZE  Set new total inventory size
  -i, --inventory       Add a frankly ludicrous number of powerful items to
                        inventory (will add appropriate inventory space
                        automatically unless --size is specified, in which
                        case --size is used)
  -n, --nodlc           Don't add DLC items when using the --inventory option
  -a item[,item,...], --additem item[,item,...]
                        Add a specific item to the inventory. Can specify this
                        option multiple times, and/or specify a list of items
                        separated by commas. (Will increase inventory size
                        automatically if required.)
  -v, --verbose         Show extra information during list. Specify more than
                        once to also show a hexdump of data at the point we
                        stopped parsing.
```

The `--check` option was mostly just useful for myself, to make sure that
the util was Doing The Right Thing against all my collected SWH savegames.
See my `check_writing.sh` script for how I'd typically run that while
parsing the game.

The `--experience` option will set all unlocked characters' XP to 19,999.  The
way *SteamWorld Heist* reads this file, merely setting the XP does **not**
automatically unlock character abilities, even though they show up if you
"view abilities" from inside the game.  It looks like the part of the savegame
which actually deals with those ability unlocks is much further down in the
savegame than I cared to parse, so instead we leave a single point of XP for
the characters to acquire during a mission.  When the character levels up from
9 to 10, the game retroactively applies any abilities which were "missed."  So
you'll get all those abilities after running a single mission with the
character.

(Interestingly, it looks like you only need **one** character to officially go
through the level-up process for **all** characters to receive any ability
unlocks they might have missed.  So I could theoretically rewrite `--experience`
a bit so that it sets all but one unlocked characters to 20000, and then
just left one at 19999, to serve as the "unlock" character.  In the end I think
that'd be more confusing than what we do now, though, so I'm going to leave it.)

The `--inventory` option adds a ridiculous number of items to your inventory.
By default these will include items from the *SteamWorld Heist: The Outsider*
DLC.  You can specify `--nodlc` to omit these items, if you don't actually have
that DLC.  It currently adds a total of 108 items if DLC items are included,
and 96 if not.  Basically enough to allow you to use just about any combination
of gear imaginable with all four characters active (though apart from Radioactive
Bullets, the most of any one item in there is four).

You can also add individual items "by hand" with the `--additem` option.  You can
specify the `--additem` argument multiple times, and/or specify a comma-separated
list of items, for example:

    swh_savegame.py savegame_000.dat -o savegame_001.dat -a smg_09 -a handgun_09,rpg_09

... would add the items `smg_09`, `handgun_09`, and `rpg_09`.  For many of the items,
you can get a feel for what the names are just by using `--list` to see what items are
in your current inventory.  Most weapons in the game run from `01` to `09`.  To get
a complete list, find the file `Bundle/data01.impak` in the *SteamWorld Heist* game
directory (there's also one in the DLC directories).  That's just a regular zipfile,
despite the `impak` extension - if you unzip it, look at the resulting files
`Definitions/weapons.xml` and `Definitions/utilities.xml`.

Notes on the Savegame Format
----------------------------

I'm quite sure that the bits of the file supported directly by arguments here
are being processed properly, but I've not really played around with any of the
other data in the file, so I'm really not sure exactly how any of the rest
works.  I think it would be trivial to allow the utility to add new hats, for
instance, but I'm not sure if hats and items share a common ID pool, or if hats
are separate.  I also suspect it wouldn't be difficult to unlock characters
before they're meant to be unlocked (when not in NG+, of course), but I didn't
bother trying.

If you take a look at a savegame in a hex editor and look near the end, you'll
see what looks like lists of abilities which have been unlocked for characters,
spread out a little bit among other bits of data.  I suspect that's where I'd have
to parse out the file if I wanted to be able to "properly" unlock character XP,
rather than doing the 19999 trick as `--experience` does, and letting the game
do the unlocks.  There's a lot of data between where I stopped and where that bit
of the file is, though, and I didn't care enough to slog through it.

*SteamWorld Heist* saves the game midlevel if you quit while on a mission, and
it looks like all that level data is after the point where I stopped processing,
since it seemed to load just fine with `--check`.  I bet if that bit of the savegame
is parsed out, you could play a lot of games with item/enemy placement, etc, on
active levels.  Not really worth figuring out, though, IMO.

As I mentioned far above, I've verified that this works against the collection
of savegames that I saved, though I am quite sure that there are savegames out there
which this utility'll choke on currently.  My main two groups were a handful of
savegames right near the end of the game (one just before the final level, basically,
and a couple more after I bought the DLC and went through the DLC levels), and then
a bunch more right near the very beginning of the game (including one NG+ savegame).
I'm going to be playing through the game again pretty soon most likely, though, and
I'll be checking those savegames against the utility as I go, to see what other edge
cases I've missed.

In particular, I suspect that many of the places in the savegame loading routine where
I'm asserting that various numbers are 0x00 are actually supposed to be lists of data
which just happen to have zero elements on all my savegames.  On the couple of updates
that I've had to do since figuring out what I cared about, that's been what's introduced
problems.

And, of course, all of the data marked with "unknown" as varnames are basically just
guesses as to what size data they are.  When presented with five unknown bytes, do you
read it as a uint32 followed by a byte, or the other way around?  Or two uint16s and
a byte?  Lots of options, certainly, and I'm sure that I've gotten it wrong in a number
of places.

If you do run across a savegame which this can't open, do feel free to open up an
Issue here, and attach the savegame, and I'll see if I can patch it up.
