# Hero Protocol

heroprotocol is a reference Python library and standalone tool to decode Heroes of the Storm replay files into Python data structures.

Currently heroprotocol can decode these structures and events:
* replay header
* game details
* replay init data
* game events
* message events
* tracker events

heroprotocol can be used as a base-build-specific library to decode binary blobs, or it can be run as a standalone tool to pretty print information from supported replay files.

Note that heroprotocol does not expose game balance information or provide any kind of high level analysis of replays; it's meant
to be just the first tool in the chain for your data mining application.

# How to use
 * install python 2.x (https://www.python.org/downloads/release/python-2711/) 
 ```
 usage: heroprotocol.py [-h] [--gameevents] [--messageevents] [--trackerevents]
                       [--attributeevents] [--header] [--details] [--initdata]
                       [--stats]
                       replay_file

positional arguments:
  replay_file        .SC2Replay file to load

optional arguments:
  -h, --help         show this help message and exit
  --gameevents       print game events
  --messageevents    print message events
  --trackerevents    print tracker events
  --attributeevents  print attributes events
  --header           print protocol header
  --details          print protocol details
  --initdata         print protocol initdata
  --stats            print stats
  ```

# Supported Versions

heroprotocol supports all Hereos of the Storm replay files that were written with retail versions of the game. The current plan is to support all future publicly released versions, including public betas.

# Tracker Events

Some notes on tracker events:
* Convert unit tag index, recycle pairs into unit tags (as seen in game events) with protocol.unit_tag(index, recycle)
* Interpret the NNet.Replay.Tracker.SUnitPositionsEvent events like this:

```python
    unitIndex = event['m_firstUnitIndex']
    for i in xrange(0, len(event['m_items']), 3):
        unitIndex += event['m_items'][i + 0]
        x = event['m_items'][i + 1] * 4
        y = event['m_items'][i + 2] * 4
        # unit identified by unitIndex at the current event['_gameloop'] time is at approximate position (x, y)
```
* Only units that have inflicted or taken damage are mentioned in unit position events, and they occur periodically with a limit of 256 units mentioned per event.
* NNet.Replay.Tracker.SUnitInitEvent events appear for units under construction. When complete you'll see a NNet.Replay.Tracker.SUnitDoneEvent with the same unit tag.
* NNet.Replay.Tracker.SUnitBornEvent events appear for units that are created fully constructed.
* You may receive a NNet.Replay.Tracker.SUnitDiedEvent after either a UnitInit or UnitBorn event for the corresponding unit tag.
* In NNet.Replay.Tracker.SPlayerStatsEvent, m_scoreValueFoodUsed and m_scoreValueFoodMade are in fixed point (divide by 4096 for integer values). All other values are in integers.
* There's a known issue where revived units are not tracked, and placeholder units track death but not birth.

# License

Copyright (c) 2015 Blizzard Entertainment

Open sourced under the MIT license. See the included LICENSE file for more information.

# Acknowledgements

The standalone tool uses [mpyq](https://github.com/arkx/mpyq/) to read mopaq files.

Thanks to David Joerg and Graylin Kim of [GGTracker](http://www.ggtracker.com) for design feedback and beta-testing of the s2protocol library that heroprotocol is based upon.
