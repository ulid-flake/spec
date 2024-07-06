Inspired by [ULID](https://github.com/ulid/spec) and [Twitter's Snowflake](https://blog.x.com/engineering/en_us/a/2010/announcing-snowflake). Ulid-Flake has a compact `64-bit` size, featuring a `1-bit` sign bit, a `43-bit` timestamp, and a `20-bit` randomness. Also provide a scalable version using the last `5-bit` as scalability.

<h1 align="left">
	<img width="240" src="logo.png" alt="ulid-flake">
</h1>

# Ulid-Flake: A `64-bit` ULID variant featuring Snowflake

## Introduction

Ulid-Flake aims to create a compact and efficient identifier system suitable for `slow-small-season-business` environments. These identifiers are optimized for environments where IDs are generated roughly once per second, with occasional bursts of multiple IDs per millisecond during peak times.

### What is a `slow-small-season-business`?

- Average ID generation rate: 1 ID per second.
- Occasional bursts: Multiple IDs generated within the same millisecond during peak periods.
- Single database setup: Typically no distributed system, single DB use.
- Multiple pods setup occasionally: Even with multiple pods setup, the number of pod replicas that used can still be counted with fingers.
- Compact and readable IDs: Prefer shorter and more readable strings for user interactions.
- Predictability vs monotonicity: Prioritize difficulty in guessing IDs over strict monotonic order.
- Time span: Usable over a 100-year time frame.

herein is proposed Ulid-Flake:

```go
ulidflake.New() // 00CMXB6TAK4SA
ulidflake.New().Int() // 14246757444195114
```

- Compatibility
  - `64-bit` compatibility with long int (`int64`, `bigint`)
- Capacity
  - Over 1 million unique Ulid-Flakes per millisecond with minimum `+1` increment for stand-alone version
  - Over 32 thousands unique Ulid-Flakes per millisecond with minimum `+1` increment for scalable version
- Scalability
  - 32 configurations for scalability using a distributed system for scalable version
- Sorting
  - Lexicographically sortable
- Encoding
  - Canonically encoded as a `13-character` string
  - Optionally can also be safely provided as a long integer (`int64`, `bigint`)
  - Uses Crockford's Base32 for better efficiency and readability (5 bits/character)
  - Case insensitive
  - No special characters (URL safe)
- Monotonicity and randomness
  - Monotonic sort order correctly detects and handles IDs generated in the same millisecond
  - Prefer `+n` entropy rather than `+1` for randomness incrementation, adding difficulty in guessing IDs

## Implementations in all kinds of languages

From ourselves and the community! 🎉

| Language | Author | Binary Implementation |
| -------- | ------ | --------------------- |
| C | waiting for you! | |
| C++ | waiting for you! | |
| C# | waiting for you! | |
| Clojure | waiting for you! | |
| COBOL | waiting for you! | |
| D | waiting for you! | |
| Dart | waiting for you! | |
| Elixir | waiting for you! | |
| Erlang | waiting for you! | |
| F# | waiting for you! | |
| Fortran | waiting for you! | |
| [Go](https://github.com/abailinrun/ulid-flake-go) | [abailinrun](https://github.com/abailinrun) | ✅ |
| Haskell | waiting for you! | |
| Java | waiting for you! | |
| JavaScript | waiting for you! | |
| Kotlin | waiting for you! | |
| Lisp | waiting for you! | |
| Lua | waiting for you! | |
| Objective-C | waiting for you! | |
| Perl | waiting for you! | |
| PHP | waiting for you! | |
| [Python](https://github.com/ulid-flake/python) | [ulid-flake](https://github.com/ulid-flake) | ✅ |
| R | waiting for you! | |
| Ruby | waiting for you! | |
| Rust | waiting for you! | |
| Scala | waiting for you! | |
| Scheme | waiting for you! | |
| Swift | waiting for you! | |
| Zig | waiting for you! | |
| more... | waiting for you! | |


## Specification

Below is the default stand-alone version specification of Ulid-Flake.

<img width="600" alt="ulid-flake-stand-alone" src="https://github.com/ulid-flake/spec/assets/38312944/37d44c3f-1937-4c2e-b7ec-e7c0f0debe25">

*Note: a `1-bit` sign bit is included in the timestamp.*

```text
Stand-alone version (default):

 00CMXB6TA      K4SA

|---------|    |----|
 Timestamp   Randomness
   44-bit      20-bit
   9-char      4-char
```

Also, a scalable version is provided for distributed system using purpose.

<img width="600" alt="ulid-flake-scalable" src="https://github.com/ulid-flake/spec/assets/38312944/e306ebd9-9406-436f-b6cd-a1004745f1b0">

*Note: a `1-bit` sign bit is included in the timestamp.*

```
Scalable version (optional):

 00CMXB6TA      K4S       A

|---------|    |---|     |-|
 Timestamp   Randomness  Scalability
   44-bit      15-bit    5-bit
   9-char      3-char    1-char
```

### Components

Total `64-bit` size for compatibility with common integer (`long int`, `int64` or `bigint`) types.

**Timestamp**
- The first `1-bit` is a sign bit, always set to 0.
- Remaining `43-bit` timestamp in millisecond precision.
- Custom epoch for extended usage span, starting from `2024-01-01T00:00:00.000Z`.
- Usable until approximately `2302-09-27` AD.

**Randomness**
- `20-bit` randomness for stand-alone version. Provides a collision resistance with a p=0.5 expectation of 1,024 trials. (not much)
- `15-bit` randomness for scalable version.
- Initial random value at each millisecond precision unit.
- adopt a `+n` bits entropy incremental mechanism to ensure uniqueness without predictability.

**Scalability (Scalable version ony)**
- Provide a `5-bit` scalability for distributed system using purpose.
- total 32 configurations can be used.

### Sorting

The left-most character must be sorted first, and the right-most character sorted last, ensuring lexicographical order.
The default ASCII character set must be used.

When using the stand-alone version strictly in a stand-alone environment, or using the scalable version in both stand-alone or distributed environment, sort order is guaranteed within the same millisecond. however, when using the stand-alone version in a distributed system, sort order is not guaranteed within the same millisecond.

*Note: within the same millisecond, sort order is guaranteed in the context of an overflow error could occur.*

### Canonical String Representation

```text
Stand-alone version (default):

tttttttttrrrr

where
t is Timestamp (9 characters)
r is Randomness (4 characters)
```

```text
Scalable version (optional):

tttttttttrrrs

where
t is Timestamp (9 characters)
r is Randomness (3 characters)
s is Scalability (1 characters)
```

#### Encoding

Crockford's Base32 is used as shown. This alphabet excludes the letters I, L, O, and U to avoid confusion and abuse.

```
0123456789ABCDEFGHJKMNPQRSTVWXYZ
```

### Optional Long Int Representation

```text
1234567890123456789

(with a maximum 13-character length in string format)
```

### Monotonicity and Overflow Error Handling

#### Randomness

When generating a Ulid-Flake within the same millisecond, the `randomness` component is incremented by a `n-bit` entropy in the least significant bit position (with carrying).
Thus, comparing just incremented `1-bit` one time, the incremented `n-bit` mechanism cloud lead to an overflow error sooner.

when the generation is failed with overflow error, it should be properly handled in the application to wait and create a new one till the next millisecond is coming. The implementation of Ulid-Flake should just return the overflow error, and leave the rest to the application.

#### Timestamp and Over All

Technically, a `13-character` Base32 encoded string can contain 65 bits of information, whereas a Ulid-Flake must only contain 64 bits. Further more, there is a `1-bit` sign bit at the beginning, only 63 bits are actually carrying effective information. Therefore, the largest valid Ulid-Flake encoded in Base32 is `7ZZZZZZZZZZZZ`, which corresponds to an epoch time of `8,796,093,022,207` or `2^43 - 1`.

Any attempt to decode or encode a Ulid-Flake larger than this should be rejected by all implementations and return an overflow error, to prevent overflow bugs.

### Binary Layout and Byte Order

The components are encoded as 16 octets. Each component is encoded with the Most Significant Byte first (network byte order).

```
Stand-alone version (default):

 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                      32_bit_int_time_high                     |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| 12_bit_uint_time_low  |          20_bit_uint_random           |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

```
Scalable version (optional):

 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                      32_bit_int_time_high                     |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| 12_bit_uint_time_low  |      15_bit_uint_random     | 5_bit_s |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

# Acknowledgments

[ULID](https://github.com/ulid/spec)

[Twitter's Snowflake](https://blog.x.com/engineering/en_us/a/2010/announcing-snowflake)

```
██╗░░░██╗██╗░░░░░██╗██████╗░░░░░░░███████╗██╗░░░░░░█████╗░██╗░░██╗███████╗
██║░░░██║██║░░░░░██║██╔══██╗░░░░░░██╔════╝██║░░░░░██╔══██╗██║░██╔╝██╔════╝
██║░░░██║██║░░░░░██║██║░░██║█████╗█████╗░░██║░░░░░███████║█████═╝░█████╗░░
██║░░░██║██║░░░░░██║██║░░██║╚════╝██╔══╝░░██║░░░░░██╔══██║██╔═██╗░██╔══╝░░
╚██████╔╝███████╗██║██████╔╝░░░░░░██║░░░░░███████╗██║░░██║██║░╚██╗███████╗
░╚═════╝░╚══════╝╚═╝╚═════╝░░░░░░░╚═╝░░░░░╚══════╝╚═╝░░╚═╝╚═╝░░╚═╝╚══════╝
```
