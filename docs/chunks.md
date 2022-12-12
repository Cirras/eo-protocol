# Chunked Reading

## Background
In the official EO client and server, packets and data files are typically just treated as ansi strings.
The notable exception is the "chunked" reading mode, typically used for reading data that contains multiple variable-sized fields.

### How it works
- Chunked reading mode gets activated.
  - For our example, we'll call this function `DataReader::StartChunking`.
  - The data string is split on `0xFF` bytes, which are called `break` bytes in this context.
- Chunks of data are requested sequentially, with the next string chunk being returned each time.
  - For our example, we'll call this function `DataReader::GetChunk`.

#### Example:
``` cpp
// This is an approximation of how the packet chunking looks in the official data reader.
//
// Let's read a FOO_BAR packet containing the following data:
//   char reply_code
//   break
//   string name
//   break
//   int session_id
//   short class_id
void FooBar(DataReader* reader)
{
    // Read the reply_code directly from the packet data. This is how data is read in a standard case.
    unsigned char reply_code = DecodeNumber(reader->data.SubString(1, 1));

    // Activate chunked reading.
    reader->StartChunking();

    // discard the first chunk containing the reply code.
    reader->GetChunk();

    // Get the entire second chunk for the name field.
    AnsiString name = reader->GetChunk();

    // Get the third chunk, containing 2 number fields.
    AnsiString numbers = reader->GetChunk();

    // Read number fields from the 3rd chunk.
    unsigned int session_id = DecodeNumber(numbers.SubString(1, 4));
    unsigned short class_id = DecodeNumber(numbers.SubString(4, 2));
}
```

> **Notes:**
> - Indexing of the C++ builder `AnsiString` type is 1-based.
>   - See: [System.AnsiString](https://docwiki.embarcadero.com/Libraries/en/System.AnsiString)
## Peculiarities
There are two important peculariarities in this chunked reading method.

### 1. Over-read
The result of overreading past a chunk (or the end of the input data) is a truncated ansi string.
- For a string field, this would just mean truncation.
- For numeric fields, the missing bytes are effectively 0.

> **Implementation notes:**
> - If the end of input data is reached...
>   - Missing bytes should be treated as `0x00` when reading unencoded numbers.
>   - Missing bytes should be treated as `0xFE` when reading encoded numbers.
>   - Strings should be truncated.
> - When chunked reading is enabled...
>   - `0xFF` bytes may not be consumed until an expected `<break>` is reached in the deserialization logic.
>   - If `0xFF` is reached and hasn't been consumed yet, the end-of-input rules should be followed.

#### Example:
``` cpp
// Expected data:
//   int foo
//   break
//   short bar
//
// Actual data:
//   break
//   char = 123
void TruncatedNumbers(DataReader* reader)
{
    // Activate chunked reading.
    reader->StartChunking();

    // This will return an empty string.
    AnsiString foo_chunk = reader->GetChunk();

    // This is equivalent to decoding [0xFE, 0xFE, 0xFE, 0xFE], so the result is 0.
    unsigned int foo = DecodeNumber(foo_chunk.SubString(1, 4));

    // This will return "|", which is [0x7C].
    AnsiString bar_chunk = reader->GetChunk();

    // This is equivalent to decoding [0x7C, 0xFE, 0xFE, 0xFE], so the result is 123.
    unsigned short bar = DecodeNumber(bar_chunk.SubString(1, 2));
}
```

### 2. Double-read
As shown with `reply_code` in the `FooBar` example, the official software sometimes mixes "standard" reading with chunked reading.

The way this usually looks is:
- Some fields are read directly from the data string.
- Chunked reading is activated.
- The first chunk is discarded.
- The rest of the chunks are read via chunked reading.

This can cause surprising and unexpected behavior in the official data reader.

> **Implementation notes:**
> - When the first `<chunked>` section is reached...
>   - If the first expected element is a `<break>`, the reader should seek to the first `0xFF` byte in the input data before consuming it.

#### Example:
``` cpp
// Expected data:
//   int foo
//   break
//   char bar
//   short baz
//
// Actual data:
//   break
//   char = 123
//   short = 12345
void DoubleRead(DataReader* reader)
{
    // Because the foo int was not sent in the actual data, we're unexpectedly reading into the second chunk.
    // The following break byte, char field, and short field are read as if they were one int.
    // This ends up being [0xFF, 0x7C, 0xCA, 0x31], which decodes to 790222478.
    int foo = DecodeNumber(foo_chunk.SubString(1, 4));

    // Activate chunked reading.
    reader->StartChunking();

    // Discard the first chunk.
    // Intended to discard the foo int, but really just discards an empty chunk.
    reader->GetChunk();

    // This will return "|Ê1", which is [0x7C, 0xCA, 0x31].
    AnsiString numbers = reader->GetChunk();

    // [0x7C, 0xFE, 0xFE, 0xFE] decodes to 123.
    unsigned char bar = DecodeNumber(bar_chunk.SubString(1, 1));

    // [0xCA, 0x31, 0xFE, 0xFE] decodes to 12345.
    unsigned short baz = DecodeNumber(bar_chunk.SubString(2, 2));
}
```

## A practical example

The official client and server *rely* on the aforementioned peculiarities for correct behavior.

A prominent example (and good litmus test for a correct double-read implementation) is the
`PLAYERS_AGREE` packet.

### The client expects...
```
char characters_count
break
array of CharacterMapInfo[characters_count] (delimited by break)
break
array of NPCMapInfo
break
array of ItemMapInfo
```

### But the server sends...
```
break
struct CharacterMapInfo
break
char 1
```

### How does this work?
You might be scratching your head at this point, wondering how this could possibly work.

Let's pull back the curtain on how a `PLAYERS_AGREE` packet is actually read by the client.
```cpp
void PlayersAgree(DataReader* reader)
{
    // Read the characters_count directly from the packet data.
    // This reads the break byte (!) and the `0xFF` byte is interpreted by the number decoder as 254.
    unsigned char characters_count = DecodeNumber(reader->data.SubString(1, 1));

    // Activate chunked reading
    reader->StartChunking();

    // Discard the first chunk.
    // Intended to discard the characters_count char, but really just discards an empty chunk.
    reader->GetChunk();

    // Read characters_count number of chunks as CharacterMapInfo records.
    for (unsigned char i = 0; i < characters_count; ++i)
    {
        AnsiString chunk = reader->GetChunk();
        if (chunk.Length() < 42)
        {
            // Any chunks under 42 bytes in length are discarded here.
            // The first CharacterMapInfo record is read correctly.
            // The second chunk is a single "char 1" value, so it's discarded.
            // The remaining 252 chunks are empty, so they're also discarded.
            continue;
        }
        ReadCharacterMapInfo(chunk);
    }

    // Read the next chunk as a non-delimited array of NPCMapInfo.
    // There is no remaining data, so this chunk is empty and npcs_count = 0.
    AnsiString npcs = reader->GetChunk();
    int npcs_count = npcs.Length() / NPCMapInfo::ByteSize();

    for (int i = 0; i < npcs_count; ++i)
    {
        int npc_pos = 1 + (i * NPCMapInfo::ByteSize());
        AnsiString npc = chunk.SubString(npc_pos, NPCMapInfo::ByteSize());
        ReadNPCMapInfo(npc);
    }

    // Read the next chunk as a non-delimited array of ItemMapInfo.
    // There is no remaining data, so this chunk is empty and items_count = 0.
    AnsiString items = reader->GetChunk();
    int items_count = items.Length() / ItemMapInfo::ByteSize();

    for (int i = 0; i < items_count; ++i)
    {
        int item_pos = 1 + (i * ItemMapInfo::ByteSize());
        AnsiString item = chunk.SubString(item_pos, ItemMapInfo::ByteSize());
        ReadItemMapInfo(item);
    }
}
```

The client ends up with the single intended `CharacterMapInfo` update, and everything _accidentally_ works out fine!

## Sanitization
Some packets treat `0xFF` bytes as data, while others treat them as meaningful `break` bytes for data chunking.

A problematic scenario presents itself:
- A client sends a string containing `0xFF` (the ÿ character) in a packet where `0xFF` is treated as data.
- The server sends that string back out to clients in a packet where `0xFF` is treated as a `break` bytes.
- The clients receiving those packets are unable to read them correctly due to the unexpected `break` bytes.

> **Implementation notes:**
> - The serializer should sanitize `0xFF` bytes when writing strings if:
>    - the packet contains a `<chunked>` section.
>    - the packet contains a `struct` field that itself contains a `<chunked>` section (transitive).
> - Since `0xFF` maps to character **ÿ** in ansi codepage 1252, it can simply be swapped to **y** (`0x79`).