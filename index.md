This page documents Ace of Spades 0.75, the last fully released version of Ace of Spades classic, and 0.76, the last publically available version of Ace of Spades Classic.

# Overview
[Ace of Spades](http://buildandshoot.com/) uses the [ENet networking
library](http://enet.bespin.org/Features.html) for all server-client
communication. The only source of protocol information was the
[pyspades](http://code.google.com/p/pyspades/) source, which was made a closed
system at the release of Ace of Spades 1.0. The reason(s) for this is unknown,
but it is speculated that [Jagex](https://en.wikipedia.org/wiki/Jagex) has
paid them off, as they now own AoS.

# Versions

Except where otherwise noted, this document applies to .75, the only
commonly played version.

The Piqueserver and OpenSpades teams have introduced a number changes that are
backwards compatible with .75. This "version" of the protocol is called 0.75.1
here

# Connection

When you connect, you must send a version number as the initial data.

Following that a client needs to send an Existing Player data packet to send its own name, team etc.

If the client does not send an Existing Player packet first, but any other packet, then the server closes the connection and seems to temporarily ban the player.

| Number | AoS version |
|--------|-------------|
| 3      | 0.75        |
| 4      | 0.76        |

## Disconnect Reasons

Whenever the connection is closed by the server, there is a reason supplied to the client in the event's data (event.data).

| Number | Reason                       |
|--------|------------------------------|
| 1      | Banned                       |
| 2      | IP connection limit exceded? |
| 3      | Wrong protocol version       |
| 4      | Server full                  |
| 10     | Kicked                       |
| 20     | Invalid Name (new in 0.75.1) |

Send this magic number as part of the `enet_host_connect(ENetHost, ENetAddress, channels, int)` function

## About Coordinates

In Ace of Spades the up-down axis is Z and it is inverted. This means 63 is water level and 0 is the highest point on a map.

# Packets

All packets start with an unsigned byte to specify their type, followed by the data for that type of packet. The size given for each packet below includes this byte.

## Data types

Generally, all fields in the Protocol are Low Endian if not specified.

The following shorthands are used in this document:

| Shorthand    | details                                                 |
| -----------: | ----------                                              |
| Byte         | 8 bits of arbitrary data. Usually accompanied by a note |
| UByte        | Unisgned 8 bit number                                   |
| LE Float     | 32bit IEEE float                                        |
| LE Uint      | 32bit unsigned integer                                  |
| CP437 String | String encoded with CP437. Usually fixed-length.        |

## Position Data
`Client <-> Server`

This packet is used to set the players position.

|-----------:|----------|
|Packet ID:  |  0       |
|Total Size: | 12 bytes |

|Field Name|Field Type|Example|Notes|
|---------:|----------|-------|-----|
|        x | LE Float |  `0`  |     |
|        y | LE Float |  `0`  |     |
|        z | LE Float |  `0`  |     |

## Orientation Data
This packet is used to set the players orientation.

|-----------:|----------|
|Packet ID   |  1       |
|Total Size: | 13 bytes |

#### Fields

|Field Name|Field Type|Example|Notes|
|---------:|----------|-------|-----|
|        x | LE Float |  `0`  |     |
|        y | LE Float |  `0`  |     |
|        z | LE Float |  `0`  |     |

## World Update (0.75)
Updates position and orientation of all players. Always sends data for 32 players, with empty slots being all 0 (position: [0,0,0], orientation: [0,0,0]).

| ----------: | -------- |
| Packet ID   | 2        |
| Total Size: | 13 bytes |

#### Fields

| Field Name                         | Field Type                        | Example | Notes                    |
|------------------------------------|-----------------------------------|---------|--------------------------|
| players positions and orientations | Array[32] of Player Position Data |         | See below table for data |

#### 'Player Position Data'

| ----------: | --------- |
| Total Size: | 769 bytes |

#### Fields

| Field Name    | Field Type | Example | Notes             |
|---------------|------------|---------|-------------------|
| x position    | LE Float   | `0`     | 0 for non-players |
| y position    | LE Float   | `0`     | 0 for non-players |
| z position    | LE Float   | `0`     | 0 for non-players |
| x orientation | LE Float   | `0`     | 0 for non-players |
| y orientation | LE Float   | `0`     | 0 for non-players |
| z orientation | LE Float   | `0`     | 0 for non-players |

## World Update (0.76)
Updates position and orientation of all players. Unlike 0.75, this only sends information for the necessary players.

| -----------: | ----------    |
| Packet ID    | 2             |
| Total Size:  | 1 + 25n bytes |

#### Fields

| Field Name                         | Field Type                                     | Example | Notes                    |
|------------------------------------|------------------------------------------------|---------|--------------------------|
| players positions and orientations | Array[] of Player Position Data, variable size |         | See below table for data |

#### 'Player Position Data'

|------------:|----------|
| Total Size: | 24 bytes |

#### Fields

| Field Name    | Field Type | Example | Notes |
|---------------|------------|---------|-------|
| player ID     | UByte      | `0`     |       |
| x position    | LE Float   | `0`     |       |
| y position    | LE Float   | `0`     |       |
| z position    | LE Float   | `0`     |       |
| x orientation | LE Float   | `0`     |       |
| y orientation | LE Float   | `0`     |       |
| z orientation | LE Float   | `0`     |       |

## Input Data
Contains the key-states of a player, packed into a byte.

| ----------- | -------- |
| Packet ID   | 3        |
| Total Size: | 3 bytes  |

#### Fields

| Field Name | Field Type | Example | Notes                                                                 |
|------------|------------|---------|-----------------------------------------------------------------------|
| player ID  | UByte      | `0`     |                                                                       |
| key states | UByte      | `0`     | Each bit in the byte represents a key, as defined in the table below. |

#### Key States:

| Placement | Key    |
| --------- | ------ |
| 1         | up     |
| 2         | down   |
| 3         | left   |
| 4         | right  |
| 5         | jump   |
| 6         | crouch |
| 7         | sneak  |
| 8         | sprint |

## Weapon Input
Contains the weapon input state(?).

|------------:|---------|
| Packet ID   | 4       |
| Total Size: | 3 bytes |

| Field Name   | Field Type | Example | Notes                                                                                        |
|--------------|------------|---------|----------------------------------------------------------------------------------------------|
| player ID    | UByte      | `0`     |                                                                                              |
| weapon input | UByte      | `0`     | The lowest bit represents the primary fire, the second lowest represents the secondary fire. |

## Hit Packet
#### Client-to-Server

Sent by the client when a hit is registered. The server should verify that this is possible to prevent abuse (such as hitting without shooting, facing the wrong way, etc).


| -----------:| ------- |
| Packet ID   | 5       |
| Total Size: | 3 bytes |

#### Fields

| Field Name    | Field Type | Example | Notes                     |
|---------------|------------|---------|---------------------------|
| player ID hit | UByte      | `0`     |                           |
| hit type      | UByte      | `0`     | See values in table below |

#### Hit Types

| Value | Type  |
| ----- | ----- |
| 0     | torso |
| 1     | head  |
| 2     | arms  |
| 3     | legs  |
| 4     | melee |

## Set HP
#### Server-to-Client

Sent to the client when hurt.


| -----------:| -------- |
| Packet ID   | 5        |
| Total Size: | 15 bytes |

| Field Name        | Field Type | Example | Notes                |
|-------------------|------------|---------|----------------------|
| HP                | UByte      | `0`     |                      |
| type              | UByte      | `0`     | 0 = fall, 1 = weapon |
| source x position | LE Float   | `0`     |                      |
| source y position | LE Float   | `0`     |                      |
| source z position | LE Float   | `0`     |                      |

## Grenade Packet
Spawns a grenade with the given information.

| ------------:| --------- |
| Packet ID    | 6         |
| Total Size:  | 30 bytes  |

#### Fields

| Field Name  | Field Type | Example | Notes |
|-------------|------------|---------|-------|
| player ID   | UByte      | `0`     |       |
| fuse length | LE Float   | `0`     |       |
| x position  | LE Float   | `0`     |       |
| y position  | LE Float   | `0`     |       |
| z position  | LE Float   | `0`     |       |
| x velocity  | LE Float   | `0`     |       |
| y velocity  | LE Float   | `0`     |       |
| z velocity  | LE Float   | `0`     |       |

## Set Tool
Sets a player's current;y equipped tool/weapon.


|------------:|---------|
| Packet ID   | 7       |
| Total Size: | 3 bytes |

#### Fields

| Field Name | Field Type | Example | Notes                        |
|------------|------------|---------|------------------------------|
| player ID  | UByte      | `0`     |                              |
| tool       | UByte      | `0`     | Tool values are listed below |

#### Tools

| Value | Type    |
| ----- | -----   |
| 0     | spade   |
| 1     | block   |
| 2     | gun     |
| 3     | grenade |

## Set Colour
Set the colour of a player's held block.

|------------:|---------|
| Packet ID   | 8       |
| Total Size: | 5 bytes |

| Field Name | Field Type | Example | Notes |
|------------|------------|---------|-------|
| player ID  | UByte      | `0`     |       |
| blue       | UByte      | `0`     |       |
| green      | UByte      | `0`     |       |
| red        | UByte      | `0`     |       |

## Existing Player
Set player's team, weapon, etc.


|------------:|---------|
| Packet ID   | 9       |
| Total Size: | depends |

#### Fields

| Field Name | Field Type                                                 | Example | Notes |
|------------|------------------------------------------------------------|---------|-------|
| player ID  | UByte                                                      | `0`     |       |
| team       | Byte                                                       | `0`     |       |
| weapon     | UByte                                                      | `0`     |       |
| held item  | UByte                                                      | `0`     |       |
| kills      | LE UInt                                                    | `0`     |       |
| blue       | UByte                                                      | `0`     |       |
| green      | UByte                                                      | `0`     |       |
| red        | UByte                                                      | `0`     |       |
| name       | [CP437](http://en.wikipedia.org/wiki/Code_page_437) String | ``      |       |

## Short Player Data
Like Existing Player, but with less information.

|------------:|---------|
| Packet ID   | 10      |
| Total Size: | 4 bytes |

#### Fields

| Field Name | Field Type | Example | Notes |
|------------|------------|---------|-------|
| player ID  | UByte      | `0`     |       |
| team       | Byte       | `0`     |       |
| weapon     | UByte      | `0`     |       |

## Move Object
Brief description.

| ----------: | -------- |
| Packet ID   | 11       |
| Total Size: | 15 bytes |

#### Fields

| Field Name | Field Type | Example | Notes       |
|------------|------------|---------|-------------|
| object id  | UByte      | `0`     |             |
| team       | UByte      | `0`     | 2 = neutral |
| x position | LE Float   | `0`     |             |
| y position | LE Float   | `0`     |             |
| z position | LE Float   | `0`     |             |

## Create Player
Brief description.

| ----------: | ------- |
| Packet ID   | 12      |
| Total Size: | depends |

#### Fields

| Field Name | Field Type                                                 | Example | Notes |
|------------|------------------------------------------------------------|---------|-------|
| player id  | UByte                                                      | `0`     |       |
| weapon     | UByte                                                      | `0`     |       |
| team       | Byte                                                       | `0`     |       |
| x position | LE Float                                                   | `0`     |       |
| y position | LE Float                                                   | `0`     |       |
| z position | LE Float                                                   | `0`     |       |
| name       | [CP437](http://en.wikipedia.org/wiki/Code_page_437) String | ``      |       |

## Block Action
Sent when a block is placed/destroyed.

| ----------: | -------- |
| Packet ID   | 13       |
| Total Size: | 15 bytes |

#### Fields

| Field Name  | Field Type | Example | Notes           |
|-------------|------------|---------|-----------------|
| player id   | UByte      | `0`     |                 |
| action type | UByte      | `0`     | See table below |
| x position  | LE Int     | `0`     |                 |
| y position  | LE Int     | `0`     |                 |
| z position  | LE Int     | `0`     |                 |


#### Fields

| Value | Type              |
| ----- | ----------------- |
| 0     | build             |
| 1     | (bullet?) destroy |
| 2     | spade destroy     |
| 3     | grenade destroy   |

## Block Line
Create a line of blocks between 2 points.

| ----------: | -------- |
| Packet ID   | 14       |
| Total Size: | 26 bytes |

| Field Name       | Field Type      | Example | Notes |
| player id        | UByte           | `0`     |       |
| start x position | LE Int          | `0`     |       |
| start y position | LE Int          | `0`     |       |
| start z position | LE Int          | `0`     |       |
| end x position   | LE Int          | `0`     |       |
| end y position   | LE Int          | `0`     |       |
| end z position   | LE Int          | `0`     |       |

## CTF State
Brief description.

| ----------: | -------- |
| Packet ID   | none     |
| Total Size: | 52 bytes |

#### Fields

| Field Name             | Field Type    | Example | Notes                                                                |
| team 1 score           | UByte         | `0`     |                                                                      |
| team 2 score           | UByte         | `0`     |                                                                      |
| capture limit          | UByte         | `0`     |                                                                      |
| intel flags            | UByte         | `0`     | bits signal if teams have intel - bit 1 for team 1, bit 2 for team 2 |
| team 1 intel location  | Location Data | `0`     | see below                                                            |
| team 2 intel location  | Location Data | `0`     | see below                                                            |
| team 1 base x position | LE Float      | `0`     |                                                                      |
| team 1 base y position | LE Float      | `0`     |                                                                      |
| team 1 base z position | LE Float      | `0`     |                                                                      |
| team 2 base x position | LE Float      | `0`     |                                                                      |
| team 2 base y position | LE Float      | `0`     |                                                                      |
| team 2 base z position | LE Float      | `0`     |                                                                      |

The intel location data is 12 bytes long. If the intel is being held, the first
byte is a UByte with the id of the holding player, then the rest are padding.
If the intel is on the ground (not being held), the data will hold three LE
Floats with its x, y and z position.

#### Fields

| Intel State         | Field Name        | Field Type   |
| ------------------- | ------------      | ------------ |
| Held                | holding player id | UByte        |
|                     | padding           | 11 bytes     |
| Dropped             | intel x position  | LE Float     |
|                     | intel y position  | LE Float     |
|                     | intel z position  | LE Float     |

This packet is not a complete packet, as it is only sent after the initial
data, where the gamemode is sent. It could be considered as part of that
initial data packet, but as what's sent varies greatly depending on the
gamemode, it is documented separately.

## Territory
Brief description.

## Object Territory
Brief description.


## TCState

| Field Name                | Field Type                          | Example | Notes
| territory count           | UByte                               | 16      | Maximum is 16 otherwise the client will crash with 'Invalid memory access'
| Array[] of territory data | LE Float, LE Float, LE Float, UByte |         | See table below

This packet is not a complete packet, as it is only sent after the initial
data, where the gamemode is sent. It could be considered as part of that
initial data packet, but as what's sent varies greatly depending on the
gamemode, it is documented separately.

## State Data
`Server-->Client`

Indicates that the map transfer is complete. Also informs the client of numerous game parameters. Be aware that CTFState or TCState may be appended to the packet after the gamemode id portion.

| ----------: | -------- |
| Packet ID   | 15       |
| Total Size: | 52 bytes |

#### Fields

| Field Name             | Field Type     | Example   | Notes                     |
| ---------------------- | -------------- | --------- | ------------------------- |
| fog (blue)             | UByte          | 0         |                           |
| fog (green)            | UByte          | 0         |                           |
| fog (red)              | UByte          | 0         |                           |
| team 1 color (blue)    | UByte          | 0         |                           |
| team 1 color (green)   | UByte          | 0         |                           |
| team 1 color (red)     | UByte          | 0         |                           |
| team 2 color (blue)    | UByte          | 0         |                           |
| team 2 color (green)   | UByte          | 0         |                           |
| team 2 color (red)     | UByte          | 0         |                           |
| team name 1            | CP437 String   | Blue      | Always 10 characters long |
| team name 2            | CP437 String   | Green     | Always 10 characters long |
| gamemode id            | UByte          | 0         | 0 for CTF, 1 for TC       |


## Kill Action
#### Server->Client

Notify the client of a player's death.

| ----------: | -------- |
| Packet ID   | 18       |
| Total Size: | 5 bytes  |

| Field Name       | Field Type | Example | Notes                 |
|------------------|------------|---------|-----------------------|
| player ID        | UByte      | 12      | Player that died      |
| killer ID        | UByte      | 8       |                       |
| kill type        | UByte      | 0       | See table below       |
| respawn time     | UByte      | 1       | Seconds until respawn |

#### Fields

| Value | Type                 |
|-------|----------------------|
| 0     | WEAPON (body, limbs) |
| 1     | HEADSHOT             |
| 2     | MELEE (spade)        |
| 3     | GRENADE              |
| 4     | FALL                 |
| 5     | TEAM\_CHANGE         |
| 6     | CLASS\_CHANGE        |

## Chat Message
#### Two-way

Reasonable limits must placed on length and frequency of chat messages.


| ----------: | -------- |
| Packet ID   | 17       |
| Total Size: | . bytes  |

| Field Name   | Field Type                                                 | Example           | Notes           |
|--------------|------------------------------------------------------------|-------------------|-----------------|
| player id    | UByte                                                      | `0`               |                 |
| Chat Type    | UByte                                                      | `0`               | See table below |
| Chat Message | [CP437](http://en.wikipedia.org/wiki/Code_page_437) String | `"join /squad 1"` |                 |



#### Fields

| Value | Type         |
|-------|--------------|
| 0     | CHAT\_ALL    |
| 1     | CHAT\_TEAM   |
| 2     | CHAT\_SYSTEM |

## Map Start (0.75)
#### Server->Client

Sent when a client connects, or a map is advanced for already existing connections.

Should be the first packet received when a client connects.

| ----------: | -------- |
| Packet ID   | 18       |
| Total Size: | 5 bytes  |

| Field Name | Field Type | Example | Notes |
|------------|------------|---------|-------|
| Map size   | Uint32     | `4567`  |       |

## Map Start (0.76)
#### Server->Client

Sent when a client connects, or a map is advanced for already existing connections.

Should be the first packet received when a client connects.


| ----------: | -------- |
| Packet ID   | 18       |
| Total Size: | 9+ bytes |

#### Fields

| Field Name | Field Type                                                 | Example       | Notes |
|------------|------------------------------------------------------------|---------------|-------|
| Map size   | Uint32                                                     | `283839`      |       |
| CRC32      | Uint32                                                     | `0x4c7ebe43`  |       |
| Map name   | [CP437](http://en.wikipedia.org/wiki/Code_page_437) String | `"pinpoint2"` |       |

## Map Chunk
#### Server->Client

Sent just after [[Ace_of_Spades_Protocol#Map_Start_(0.75) Map Start]], repeatedly until the entire map is sent.

Should always be the next sequence of packets after a [[Ace_of_Spades_Protocol#Map_Start_(0.75) Map Start]].


| ----------: | -------- |
| Packet ID   | 19       |
| Total Size: | depends  |

#### Fields

| Field Name | Field Type | Example | Notes                                                                                                                              |
|------------|------------|---------|------------------------------------------------------------------------------------------------------------------------------------|
| Map Data   | UByte      | `0`     | [DEFLATE/zlib](http://en.wikipedia.org/wiki/DEFLATE) encoded [http://silverspaceship.com/aosmap/aos_file_format.html AOS map data] |

## Player Left
#### Server->Protocol

Sent when a player disconnects.


| ----------: | -------- |
| Packet ID   | 20       |
| Total Size: | 2 bytes  |

#### Fields

| Field Name | Field Type | Example | Notes |
|------------|------------|---------|-------|
| player ID  | UByte      | `0`     |       |

## Territory Capture
#### Server->Protocol

Sent when a player captures a Command Post in Territory Control mode.

Captures have affects on the client.

| ----------: | -------- |
| Packet ID   | 21       |
| Total Size: | 5 bytes  |

#### Fields

| Field Name | Field Type | Example | Notes                           |
|------------|------------|---------|---------------------------------|
| player ID  | UByte      | `0`     |                                 |
| entity ID  | UByte      | `0`     | The ID of the CP being captured |
| winning    | UByte      | `0`     | (or losing)                     |
| state      | UByte      | `0`     | team id                         |

## Progress Bar
#### Server->Client

Display the TC progress bar.


| ----------: | -------- |
| Packet ID   | 22       |
| Total Size: | 8 bytes  |

#### Fields

| Field Name        | Field Type | Example | Notes                                                                                            |
|-------------------|------------|---------|--------------------------------------------------------------------------------------------------|
| entity ID         | UByte      | `0`     | The ID of the tent entity(?)                                                                     |
| capturing team ID | UByte      | `1`     |                                                                                                  |
| rate              | Byte       | `2`     | Used by the client for interpolation(?) One per team member capturing (minus enemy team members) |
| progress          | LE Float   | `0.5`   | In range [0,1]                                                                                   |

## Intel Capture
#### Server->Protocol

Sent when a player captures the intel, which is determined by the server.

Winning captures have affects on the client.


| ----------: | -------- |
| Packet ID   | 23       |
| Total Size: | 3 bytes  |

#### Fields

| Field Name | Field Type | Example | Notes                   |
|------------|------------|---------|-------------------------|
| player ID  | UByte      | `0`     |                         |
| winning    | UByte      | `0`     | Was the winning capture |

## Intel Pickup
#### Server->Protocol

Sent when a player collects the intel, which is determined by the server.

| ----------: | -------- |
| Packet ID   | 24       |
| Total Size: | 2 bytes  |

#### Fields

| Field Name | Field Type | Example | Notes |
|------------|------------|---------|-------|
| player ID  | UByte      | `0`     |       |

## Intel Drop
#### Server->Protocol

Sent when a player dropped the intel. This will update the intel position on the client.


| ----------: | -------- |
| Packet ID   | 25       |
| Total Size: | 13 bytes |

#### Fields

| Field Name | Field Type | Example | Notes                              |
| player ID  | UByte      | `0`     | ID of the player who dropped intel |
| x position | LE Int     | `32.0`  |                                    |
| y position | LE Int     | `32.0`  |                                    |
| z position | LE Int     | `32.0`  |                                    |

## Restock
#### Server->Protocol

Id of the player who has been restocked.

| ----------: | -------- |
| Packet ID   | 26       |
| Total Size: | 2 bytes  |

#### Fields

| Field Name | Field Type | Example | Notes                          |
|------------|------------|---------|--------------------------------|
| player ID  | UByte      | `0`     | ID of the player who restocked |

## Fog Colour
#### Server->Client

Set the colour of a player's fog.

| ----------: | -------- |
| Packet ID   | 27       |
| Total Size: | 5 bytes  |

#### Fields

| Field Name | Field Type | Example      | Notes        |
| ---------- | ---------- | ------------ | ------------ |
| fog color  | UInt       | `0h00fefefe` | BGRA encoded |

## Weapon Reload
#### Client-->Server->Protocol

Sent by the client when the player reloads their weapon, and relayed to other clients after protocol logic applied.

This has no affect on animation, but is used to trigger sound effects on the other clients.


| ----------: | -------- |
| Packet ID   | 28       |
| Total Size: | 4 bytes |

#### Fields

| Field Name   | Field Type | Example | Notes               |
|--------------|------------|---------|---------------------|
| player ID    | UByte      | `0`     | Player who reloaded |
| clip ammo    | UByte      | `0`     |                     |
| reserve ammo | UByte      | `0`     |                     |

## Change Team
#### Client-->Server-->Protocol-->[[Ace_of_Spades_Protocol#Kill_Action Kill Action]] & [[Ace_of_Spades_Protocol#Create_Player Create Player]]

Sent by the client when the player changes team. Is not relayed to all clients directly, but instead uses [[Ace_of_Spades_Protocol#Kill_Action Kill Action]] then [[Ace_of_Spades_Protocol#Create_Player Create Player]] to inform other clients of the team change.

| ----------: | -------- |
| Packet ID   | 29       |
| Total Size: | 3 bytes  |

#### Fields

| Field Name | Field Type | Example | Notes                     |
|------------|------------|---------|---------------------------|
| player ID  | UByte      | `0`     | Player who changed team   |
| Team ID    | Byte       | `0`     | See values in table below |

#### Team IDs

| Value | Type      |
|------:|-----------|
| -1    | spectator |
| 0     | blue      |
| 1     | green     |

## Change Weapon
#### Client-->Server-->Protocol-->[[Ace_of_Spades_Protocol#Kill_Action|Kill Action]] & [[Ace_of_Spades_Protocol#Change_Weapon|Change Weapon]]

Sent by the client when player changes weapon, and relayed to clients by server after filter_visibility logic applied.

Receiving clients will also be sent a preceding [[Ace_of_Spades_Protocol#Kill_Action Kill Action]] to inform them the player has died both of which are sent as reliable packets.


| ----------: | -------- |
| Packet ID   | 30       |
| Total Size: | 3 bytes  |

| Field Name | Field Type | Example | Notes                       |
| player ID  | UByte      | `0`     | Player who's changed weapon |
| Weapon ID  | UByte      | `0`     | See values in table below   |

#### Weapon ID

| Value | Type    |
|-------|---------|
| 0     | rifle   |
| 1     | smg     |
| 2     | shotgun |

## Map Cached
#### 'Client->Server'

TODO.

# Powerthirst Edition

This version adds 4 new packets, extends 2 packets, and duplicately maps 1 packet over another.

The World Update packet has up to 64 fields now instead of 32.

## Map Start (PT)
#### Server->Client

Sent when a client connects, or a map is advanced for already existing connections.

Should be the first packet received when a client connects.

The version must exist and be >= 1 (and <= the client's Powerthirst proto version or otherwise the client will refuse to connect) to enable certain Powerthirst features such as long-name support.


| ----------: | -------- |
| Packet ID   | 18       |

| Field Name | Field Type | Example | Notes |
|------------|------------|---------|-------|
| Map size   | Uint32     | `4567`  |       |
| PT version | UInt32     | `4`     |       |

## Map Chunk (PT)
#### Server->Client

This is just a remapping of the [[Ace_of_Spades_Protocol#Map_Chunk Map Chunk]] packet to 2 packets back to stop vanilla clients from connecting.

| ----------: | -------- |
| Packet ID   | 17       |
| Total Size: | 9 bytes  |

#### Fields

| Field Name | Field Type | Example | Notes                                                                                                                               |
|------------|------------|---------|-------------------------------------------------------------------------------------------------------------------------------------|
| Map Data   | UByte      | `0`     | [DEFLATE/zlib](http://en.wikipedia.org/wiki/DEFLATE) encoded [AOS map data](http://silverspaceship.com/aosmap/aos_file_format.html) |

## Script Begin (PT)
#### Server->Client


| ----------: | --------       |
| Packet ID   | 31             |
| Total Size: | (varies) bytes |

#### Fields

| Field Name  | Field Type                                                 | Example | Notes |
|-------------|------------------------------------------------------------|---------|-------|
| Script size | Uint32                                                     | `4567`  |       |
| Module name | [CP437](http://en.wikipedia.org/wiki/Code_page_437) String | ``      |       |

## Script Chunk (PT)
#### Server->Client

This is just a remapping of the [[Ace_of_Spades_Protocol#Map_Chunk|Map Chunk]] packet to 2 packets back to stop vanilla clients from connecting.


| ----------: | --------       |
| Packet ID   | 32             |
| Total Size: | (varies) bytes |

#### Fields

| Field Name  | Field Type | Example | Notes                                                                                                                         |
|-------------|------------|---------|-------------------------------------------------------------------------------------------------------------------------------|
| Script Data | UByte      | `0`     | [DEFLATE/zlib](http://en.wikipedia.org/wiki/DEFLATE) encoded [AngelScript source code](http://www.angelcode.com/angelscript/) |


## Script End (PT)
#### Server->Client

Once this is sent, the script is loaded.

| ----------: | --------       |
| Packet ID   | 33             |
| Total Size: | (varies) bytes |

#### Fields

| Field Name  | Field Type                                                 | Example | Notes |
|-------------|------------------------------------------------------------|---------|-------|
| Module name | [CP437](http://en.wikipedia.org/wiki/Code_page_437) String | ``      |       |

## Script Call (PT)
#### Server->Client


| ----------: | --------       |
| Packet ID   | 34             |
| Total Size: | (varies) bytes |

#### Fields

| Field Name    | Field Type                                                              | Example       | Notes                                                      |
|---------------|-------------------------------------------------------------------------|---------------|------------------------------------------------------------|
| Function name | 0-terminated [CP437](http://en.wikipedia.org/wiki/Code_page_437) String | `void main()` | Must be an AngelScript prototype, not just the name itself |
| Parameters    | See below                                                               | ``            |                                                            |

### Script Parameters
Start from after the 0-byte in the Function name string. Then, loop through these IDs:

* 0: ASP_TERM: End of parameter list.
* 1: ASP_INT: Read a 32-bit little-endian int. AngelScript type: "int"
* 2: ASP_FLOAT: Read a 32-bit little-endian single-precision float. AngelScript type: "float"
* 3: ASP_PSTRING: Read an 8-bit uint, then read that many bytes as a string (do NOT add in a terminating NUL). AngelScript type: "const string &in"

With Ace of Spades being taken over by Jagex and them dropping support for Ace of Spades classic and instead creating and selling Ace of Spades 1.0, a large part of the community has broken away and aims to keep the game they love going strong. The main community forum is [http://www.buildandshoot.com/ Build and Shoot].

In this effort, people are creating their own, open-source versions of the game, listed below. If there is a project missing, and is active, add it to this page.

If you are interested in contributing to this wiki, feel free to do so.

# Discussion
You can discuss development for Ace of Spades classic in #aos.development on [irc://irc.quacknet.org/#aos.development QuackNet] ([http://webchat.quacknet.org/?channels=%23aos.development webchat]).

# Classic Projects

## Client
* [http://www.buildandshoot.com/viewtopic.php?f=5&t=74 Cube Root] ([https://github.com/RootDynasty/cuberoot github])
* [[Iceball]] ([https://github.com/iamgreaser/buldthensnip github])
* [[Voxlap Port]] ([https://github.com/Ericson2314/Voxlap github])

## Server
* [[Jack of Spades]] ([https://github.com/rakiru/Jack-of-Spades github])
* [http://code.google.com/p/pyspades/ pyspades] (<strike>[http://code.google.com/p/pyspades/ google code]</strike>[https://github.com/infogulch/pyspades github])
* [http://code.google.com/p/pysnip/ pysnip] ([http://code.google.com/p/pysnip/ google code])

## Other
* [[VoxelAuth]]

# Resources
* [[Ace of Spades Protocol]] - An attempt at fully documenting the Ace of Spades 0.75 network protocol
* [http://aoswiki.rakiru.com/webpages/index.html Misc Utils] - Online utils for Ace of Spades, such as aos:// address <=> IP converter
* [http://mystaddict.tlayeh.com/Computer%20Camp/Slab6/slab6.txt KVX File Format Specification] - A mirror of the readme for Slab6 which contains the .kvx file format, the format that the AoS model format is based on
* [http://silverspaceship.com/aosmap/aos_file_format.html VXL File Format Specification] - A description of the .vxl file format, the format used for AoS maps
* [http://enet.bespin.org/ ENet] - The networking library used by Ace of Spades
* [http://aoswiki.rakiru.com/webpages/playercount.php Playercount data] - Data log of 0.x and 1.0 player count
* [http://aoswiki.rakiru.com/webpages/playercount.html Playercount graph] - A line graph of the above data.
