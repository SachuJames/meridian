# tickvault

A time-series store for market tick data, in Rust. Trades and quotes go in
as appends and come back out through time-range queries, stored in
memory-mapped columnar segments so a query only touches the columns and
the time ranges it actually asks for.

## Design sketch

- Append-only segments. A writer fills a segment, seals it, and it becomes
  immutable; queries memory-map sealed segments read-only.
- Columnar layout. Timestamps, prices, sizes, and flags each get their own
  run inside a segment, compressed independently.
- Gorilla-style compression. Timestamps use delta-of-delta encoding,
  prices use XOR against the previous value, the usual tricks from the
  Gorilla paper, adapted to integer tick prices.
- A sparse per-segment index (first and last timestamp, min and max price)
  so time-range queries skip whole segments without opening them.

## Status

Early days: segment format and the compression codecs come first, then the
writer, then the query path and a benchmark against Parquet on the same
data.

## License

MIT
