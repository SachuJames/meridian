# meridian

A time-series store for market tick data, written in Rust.

Trades and quotes go in as appends and come back out through time-range
queries. Data lives in memory-mapped columnar segments, so a query reads
only the columns and the time ranges it actually asks for, nothing else.

Tick data has a particular shape: it arrives in order, most values barely
change from one tick to the next, and it gets read back in slices of time.
meridian is built around that shape instead of fighting it.

## How it works

- **Append-only segments.** A writer fills a segment, seals it, and it
  becomes immutable. Queries memory-map sealed segments read-only.
- **Columnar layout.** Timestamps, prices, sizes, and flags each get their
  own run inside a segment, compressed independently.
- **Gorilla-style compression.** Timestamps use delta-of-delta encoding,
  prices use XOR against the previous value. The usual tricks from the
  Gorilla paper, adapted to integer tick prices.
- **Sparse segment index.** Each segment records its first and last
  timestamp plus its min and max price, so a time-range query can skip
  whole segments without opening them.

## Status

Early. The segment format and the compression codecs come first, then the
writer, then the query path, and a benchmark against Parquet on the same
data.

## License

MIT
