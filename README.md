# Xoroshiro64StarSIMD

Header-only C++20 SIMD implementation of **xoroshiro64\*** (a fast small-state PRNG by David Blackman and Sebastiano Vigna), specialized for generating batches of 32-bit values efficiently.

This repo provides:
- `XoroshiroRNG`: a vectorized generator that advances multiple xoroshiro64\* streams in parallel.
- Batch APIs returning `std::array<uint32_t, BATCH_SIZE>` or `std::array<float, BATCH_SIZE>`.
- Fill APIs for writing aligned output directly to memory.

# Bench Results

```
scalar            : 0.145299 s  (1376.48 M u32/s)  sum=429503499343380162
simd(batch)       : 0.053677 s  (3726.00 M u32/s)  sum=429486373366956903
```

Notes: <br>
The xoroshiro-family generators like this one are **not** cryptographically secure so don't use it for such.
My implementation modifies the advance function to mildly mix across pairs of streams. 
After this it performed better on TestU01 passing the normal Crush but feel free to disable it by setting the default structural type on `advance()` to false.
The fill APIs use asserts to check memory is properly aligned so this won't be checked on release builds.
Using `alignas(REGISTER_BYTE_SIZE) std::vector etc` requires the vector but not its heap allocated resource to be aligned so be wary of that.

## Requirements / Suggestions

- x86/x86_64 CPU and a compiler that supports Intel/AMD SIMD intrinsics.
- Compile with at least AVX enabled (`-mavx`, `-mavx2`, `-mavx512f`, or `-march=native`).
	- Note: compiling without AVX support will work but a warning will be produced as all speed benefits are lost
- C++20 (minimal modification needed to make it compatible with older standards though)

## Instruction set selection

The header auto-selects the best available at compile time:

- AVX-512 (`__AVX512F__`) -> 512-bit registers -> `BATCH_SIZE = 16` floats/uint32 values
- AVX2 (`__AVX2__`) -> 256-bit registers -> `BATCH_SIZE = 8`
- AVX (`__AVX__`) -> 128-bit registers -> `BATCH_SIZE = 4`

Else it runs just a standard non vectorised version with BATCH_SIZE = 1, providing a warning at compile time.

## Usage

Include the header and pull batches:

```cpp
#include "xoroshiro64star.hpp"

int main() {
    XoroshiroRNG rng(123u);

    auto u = rng.get_batch_uint32();   // std::array<uint32_t, XoroshiroRNG::BATCH_SIZE>
    auto f = rng.get_batch_floats();   // std::array<float,    XoroshiroRNG::BATCH_SIZE> in [0, 1)
}
```

### Filling aligned buffers

`fill_aligned_uint32()` and `fill_aligned_float()` require the destination pointer to be aligned to `XoroshiroRNG::REGISTER_BYTE_SIZE`.

```cpp
#include "xoroshiro64star.hpp"
#include <array>

int main() {
    XoroshiroRNG rng;

    alignas(XoroshiroRNG::REGISTER_BYTE_SIZE)
    std::array<uint32_t, 1024> out{};

    rng.fill_aligned_uint32(out.data(), out.size());
}
```

## Benchmark

`benchmark.cpp` compares a scalar xoroshiro64\* loop against the SIMD batch API.

Build and run (AVX2 example):

```bash
g++ -O3 -std=c++20 -mavx2 -march=native -flto -funroll-loops \
	-fno-exceptions -fno-rtti -ffast-math -fomit-frame-pointer \
	benchmark.cpp -o benchmark

./benchmark 200000000
```

To target AVX-512, compile with something like `-mavx512f` (or rely on `-march=native` on machines where it enables AVX-512).

## Credits

- xoroshiro64\* algorithm: David Blackman and Sebastiano Vigna.
- SIMD implementation in this repo: see the header comment in `xoroshiro64star.hpp`.
