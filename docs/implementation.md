# Implementation Notes

The following notes are implementation details that are called out in comments within the protocol specification, but not specifically enforced (or enforceable) by code generation. These notes are provided for the purposes of accurate emulation of the original game client and server.

## General Information

- Packets must have a non-zero data length.
   - Empty packets cause the official game server to drop the client connection.
   - In the case of a packet with no meaningful data, either a constant or dummy field is provided in the specification.
   - This same restriction is *not* applied to the client receiving data from a server.
- Vestigial packets are included in the specification.
   - These packets are no longer actively used by either the client or server, but are documented for historical purposes.
   - Such packets are indicated by the term "(vestigial)" in the packet's descriptive comment.

## Server Implementation Notes

### Handling Packets

- `InitInitClientPacket`: The official server verifies the hard-coded `112` field value.
- `ItemDropClientPacket`: The official server handles the coordinate value `(255, 255)` as though dropping on the player's current position.
- `TradeRequestClientPacket`: The official server verifies the hard-coded `138` field value. The semantic meaning of this field is unknown.
- `TradeAcceptClientPacket`: The official server verifies the hard-coded `0` field value. The semantic meaning of this field is unknown.

### Sending Packets

- `PartyExpShare` (struct): If a player levels up, the official server sets the `level_up` field to the value of the player's new level.
- `ItemReplyServerPacket`: If a player levels up, the official server sets the `level_up` field to the value of the player's new level.
- `JukeboxReplyServerPacket`: The official server (erroneously) sends this packet if the player doesn't have enough gold to request a song.
- `PaperdollPingServerPacket`: The official server sends the logged in player's current class id in the `class_id` field.
- `NpcReplyServerPacket`: The official server only sends the optional fields in this packet to the NPC's attacker.
- `CastReplyServerPacket`: The official server only sends the optional fields in this packet to the NPC's attacker.
- `NpcSpecServerPacket` and `CastSpecServerPacket`: The official server only sends the optional fields in this packet to the NPC's killer.
- `NpcAcceptServerPacket` and `CastAcceptServerPacket`: The official server only sends the optional fields in this packet to the NPC's killer. Optional level-up details are only sent when a level-up event occurs.

## Client Implementation Notes

### Handling Packets

- `CharacterMapInfo` (struct): These structures are ignored if they are less than 42 bytes in length.
- `PartyExpShare` (struct): A `level_up` value greater than zero indicates a player leveled up. This value is the player's new level.
- `InitInitServerPacket`: This packet is always unencrypted. Encryption multiples and sequence values from this response should be used for subsequent communication with the server.
   - The sequence value needs to be incremented an additional time prior to normal packet processing.
- `ItemReplyServerPacket`: A `level_up` value greater than zero indicates a player leveled up. This value is the player's new level.
- `PaperdollPingServerPacket`: The official client reassigns `class_id` from the packet to the logged in player's class id with no visible impact.
- `ChestCloseServerPacket`: The official client assumes a broken chest if the packet is less than two bytes in length.
- `SpellTargetSelfServerPacket`: The official client only reads the optional `hp`/`tp` fields if the packet is greater than 12 bytes in length.
- `TradeRequestServerPacket`: The official client does not verify the hard-coded `138` field value.
- `RecoverReplyServerPacket`: The official client only reads the final two fields if a level-up event occurs. The client detects a level-up event if the packet is greater than six bytes in length.

### Sending Packets

- `GuildAcceptClientPacket`: The official client always sends a hard-coded `20202` value as the first field. It is unknown if this value is verified by the official server. The semantic meaning of this field is unknown.
- `QuestUseClientPacket`: The official client always sends quest ID `0` unless the quest id is explicitly selected via the quest switcher.