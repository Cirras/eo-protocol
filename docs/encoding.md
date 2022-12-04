# Encoding Routines
Pseudo-code for the encoding routines used with encoded integer and string types.

## Numeric limits
```c
CHAR_MAX = 253;
SHORT_MAX = CHAR_MAX * CHAR_MAX;
THREE_MAX = CHAR_MAX * CHAR_MAX * CHAR_MAX;
INT_MAX = CHAR_MAX * CHAR_MAX * CHAR_MAX * CHAR_MAX;
```

> **Note:**
>  - The largest valid value for each type is `TYPE_MAX - 1`.

## Numbers

### Encode
```c
EncodeNumber(number)
{
    value = number;

    d = 0xFE;
    if (number >= THREE_MAX)
    {
        d = value / THREE_MAX + 1;
        value %= THREE_MAX;
    }

    c = 0xFE;
    if (number >= SHORT_MAX)
    {
        c = value / SHORT_MAX + 1;
        value %= SHORT_MAX;
    }

    b = 0xFE;
    if (number >= CHAR_MAX)
    {
        b = value / CHAR_MAX + 1;
        value %= CHAR_MAX;
    }

    a = value + 1;

    return {a, b, c, d};
}
```

### Decode
```c
DecodeNumber(bytes)
{
    result = 0;
    length = min(size(bytes), 4);

    for (i = 0; i < length; ++i)
    {
        byte = bytes[i];

        if (byte == 0xFE)
            break;

        value = byte - 1;

        switch (i)
        {
            case 0:
                result += value;
                break;
            case 1:
                result += CHAR_MAX * value;
                break;
            case 2:
                result += SHORT_MAX * value;
                break;
            case 3:
                result += THREE_MAX * value;
                break;
        }
    }

    return result;
}
```

> **Note:**
>  - When encoding numbers, `0xFE` bytes are used as padding after the significant bytes.
>  - When decoding numbers, a `0xFE` byte is treated as a terminator.

## Strings
The core logic is shared between string encoding and decoding routines, with the only difference between them being a symmetrical reverse.

### Invert Characters
```c
InvertCharacters(bytes)
{
    flippy = (bytes.length is odd);

    for (i = 0; i < bytes.length; ++i)
    {
        c = bytes[i];
        f = 0;

        if (flippy)
        {
            f = 0x2E;
            if (c >= 0x50)
            {
                f *= -1;
            }
        }

        if (c >= 0x22 && c <= 0x7E)
        {
            bytes[i] = 0x9F - c - f;
        }

        flippy = !flippy;
    }
}
```

### Encode
```c
EncodeString(bytes)
{
    InvertCharacters(bytes);
    Reverse(bytes);
    return bytes;
}
```

### Decode
```c
DecodeString(bytes)
{
    Reverse(bytes);
    InvertCharacters(bytes);
    return bytes;
}
```