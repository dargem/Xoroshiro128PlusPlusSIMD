# WIP!!!

# Xoroshiro64starSIMD

This is a C++ header only library containing a SIMD implementation of the xoroshiro64* random number generator. It supports both AVX512 and AVX2 instructions. Compiling it with AVX512 will provide batches of 16 floats, while AVX2 will provide batches of 8 floats. Though this has a state space of 


<br>

Xoroshiro128+ was created by David Blackman and Sebastiano Vigna and it produces random numbers fast with excellent statistical properties while using minimal memory.
It is suitable for virtually all use cases outside cryptography.

The authors' original work:
https://xorshift.di.unimi.it/xoroshiro128plusplus.c
