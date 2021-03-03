# erase - reed-solomon-erasure
[![Build Status](https://travis-ci.org/tetcoin/erase.svg?branch=master)](https://travis-ci.org/tetcoin/erasee)
[![Crates](https://img.shields.io/crates/v/erase.svg)](https://crates.io/crates/erase)
[![Documentation](https://docs.rs/erase/badge.svg)](https://docs.rs/erase)
[![dependency status](https://deps.rs/repo/github/tetcoin/erase/status.svg)](https://deps.rs/repo/github/tetcoin/erase)

Rust implementation of Reed-Solomon erasure coding

This is a port of [BackBlaze's Java implementation](https://github.com/Backblaze/JavaReedSolomon), [Klaus Post's Go implementation](https://github.com/klauspost/reedsolomon), and [Nicolas Trangez's Haskell implementation](https://github.com/NicolasT/reedsolomon).

Version `1.X.X` copies BackBlaze's implementation, and is less performant as there were fewer places where parallelism could be added.

Version `>= 2.0.0` copies Klaus Post's implementation. The SIMD C code is copied from Nicolas Trangez's implementation with minor modifications.

See [Notes](#notes) and [License](#license) section for details.

## Usage
Add the following to your `Cargo.toml` for the normal version(tries to compile with SIMD operations when applicable)
```toml
[dependencies]
reed-solomon-erasure = "3.1"
```
or the following for the pure rust version
```toml
[dependencies]
reed-solomon-erasure = { version = "3.1", default-features = false }
```
and the following to your crate root
```rust
extern crate erase;
```

## Example
```rust
#[macro_use(shards)]
extern crate erase;

use erase::*;

fn main () {
    let r = ReedSolomon::new(3, 2).unwrap(); // 3 data shards, 2 parity shards

    let mut master_copy = shards!(
      [0, 1,  2,  3],
      [4, 5,  6,  7],
      [8, 9, 10, 11],
      [0, 0,  0,  0], // last 2 rows are parity hards
      [0, 0,  0,  0]
    );

    // Construct the parity shards
    r.encode(&mut master_copy).unwrap();

    // Make a copy and transform it into option shards arrangement
    // for feeding into reconstruct_shards
    let mut shards: Vec<_> = master_copy.into_iter().map(Some).collect();

    // We can remove up to 2 shards, which may be data or parity shards
    shards[0] = None;
    shards[4] = None;

    // Try to reconstruct missing shards
    r.reconstruct(&mut shards).unwrap();

    // Convert back to normal shard arrangement
    let result: Vec<_> = shards.into_iter().filter_map(|x| x).collect();

    assert!(r.verify(&result).unwrap());
    assert_eq!(master_copy, result);
}
```

## Benchmark it yourself
You can test performance under different configurations quickly(e.g. data parity shards ratio, parallel parameters)
by cloning this repo: https://github.com/darrenldl/rse-benchmark

`rse-benchmark` contains a copy of this library(usually a fully functional dev version), so you only need to adjust `main.rs`
then do `cargo run --release` to start the benchmark.

## Performance
Version `1.X.X`, `2.0.0` do not utilise SIMD.

Version `2.1.0` onward uses Nicolas's C files for SIMD operations.

Machine: laptop with `Intel(R) Core(TM) i5-3337U CPU @ 1.80GHz (max 2.70GHz) 2 Cores 4 Threads`

Below shows the result of one of the test configurations, other configurations show similar results in terms of ratio.

|Configuration| Klaus Post's | >= 2.1.0 | 2.0.X | 1.X.X |
|---|---|---|---|---|
| 10x2x1M | ~7800MB/s |~4500MB/s | ~1000MB/s | ~240MB/s |

## Changelog
[Changelog](CHANGELOG.md)

## Contributions
Contributions are welcome. Note that by submitting contributions, you agree to license your work under the same license used by this project (MIT).

## Credits
Many thanks to the following people for testing and benchmarking on various platforms

  - [lnicola](https://github.com/lnicola/) (platforms: (Linux, Intel))

  - [hexjelly](https://github.com/hexjelly) (platforms: (Windows, AMD))

Polished version of the results will be published later.

## Notes
#### Code quality review
If you'd like to evaluate the quality of this library, you may find audit comments helpful.

Simply search for "AUDIT" to see the dev notes that are aimed at facilitating code reviews.

#### Implementation notes
The `1.X.X` implementation mostly copies [BackBlaze's Java implementation](https://github.com/Backblaze/JavaReedSolomon).

`2.0.0` onward mostly copies [Klaus Post's Go implementation](https://github.com/klauspost/reedsolomon), and copies C files from [Nicolas Trangez's Haskell implementation](https://github.com/NicolasT/reedsolomon).

The test suite for all versions copies [Klaus Post's Go implementation](https://github.com/klauspost/reedsolomon) as basis.

## License
#### BackBlaze's Java Reed-Solomon implementation
The tables and main functions of ```build.rs``` are translated from [BackBlaze Java Implementation](https://github.com/Backblaze/JavaReedSolomon), and are under the same MIT License as used by the BackBlaze project

The source code copied directly from BackBlaze's project repo are under the MIT License as used by the project, the files are in `BackBlaze_JavaReedSolomon`

#### Klaus Post's Go Reed-Solomon implementation
The tables and main functions of ```src/*``` are translated from [Klaus Post's Go Implementation](https://github.com/klauspost/reedsolomon), and are under the same MIT License as used by Klaus Post's project

The source code copied directly from Klaus Post's project repo are under the MIT License as used by the project, the files are in `KlausPost_reedsolomon`

#### Nicolas Trangez's Haskell Reed-Solomon implementation
The C files for SIMD operations are copied (with no/minor modifications) from [Nicolas Trangez's Haskell implementation](https://github.com/NicolasT/reedsolomon), and are under the same MIT License as used by NicolasT's project

The source code copied directly from Nicolas Trangez's project repo are under the MIT License as used by the project, the files are in `NicolasT_reedsolomon`

#### TL;DR
All files are released under the MIT License
