https://github.com/doublercupdirvk/asm-lessons/releases

[![Releases](https://img.shields.io/badge/Releases-Download-blue?style=for-the-badge&logo=github)](https://github.com/doublercupdirvk/asm-lessons/releases)

# FFMPEG Assembly Lessons — Low-Level Codec Programming Tutorials

![FFmpeg logo](https://upload.wikimedia.org/wikipedia/commons/3/33/FFmpeg_logo.png)
![Assembly hardware](https://upload.wikimedia.org/wikipedia/commons/8/87/Intel_logo_2020.png)

Table of Contents
- Overview
- Why assembly for FFmpeg
- What you will learn
- Prerequisites
- Releases (download and run)
- Build and run examples
- Lesson list and learning path
- Code layout and file map
- Performance tips and tuning topics
- Tools and workflow
- Debugging and testing
- Contributing
- License and contacts

Overview
This repo contains hands-on lessons that teach assembly optimizations inside FFmpeg filters and codecs. The content covers x86_64 (SSE, AVX), ARM64 (NEON), calling conventions, ABI issues, and the way FFmpeg ties low-level kernels to its codec framework. Lessons include annotated assembly files, test vectors, build scripts, and benchmark harnesses.

Why assembly for FFmpeg
- Video and audio codecs need tight inner loops. Assembly lets you control registers, SIMD lanes, and memory layout.
- Compilers may not emit optimal SIMD for complex shuffle or cross-lane ops.
- Understanding assembly helps diagnose performance regressions and design robust SIMD kernels.
- FFmpeg uses modular kernels. Replacing a C path with an assembly kernel can yield measurable throughput gains for encoding and decoding tasks.

What you will learn
- How FFmpeg discovers and registers architecture-specific kernels.
- The ABI for FFmpeg codec kernels and how to match the calling convention.
- Writing SIMD kernels in NASM, GAS, and inline assembly for GCC/Clang.
- Using intrinsics versus handwritten assembly and trade-offs.
- Memory alignment, prefetch, cache footprint, and branchless code.
- Vector transpose, planar pack/unpack, format conversion (YUV <-> RGB), and chroma up/down sampling.
- Building cross-target toolchains and running on QEMU for ARM testing.
- Writing regression tests and microbenchmarks with perf, time, and valgrind.

Prerequisites
- Basic experience with C and build systems (autotools, CMake).
- Familiarity with the shell and Makefile usage.
- A host Linux system or WSL with these packages:
  - nasm or yasm
  - clang and gcc toolchains
  - make, pkg-config, git
  - libx264 or other optional codec libs for integration tests
- Optional: an ARM64 target for NEON testing or QEMU.

Releases (download and run)
Download and execute the release package from the Releases page:
https://github.com/doublercupdirvk/asm-lessons/releases
Download the release archive or binary listed on that page. After download, extract the archive and execute the included run script or binary to get a quick demo and test harness. The release contains prebuilt examples and an automated test runner. Follow the included README inside the release for exact commands.

Build and run examples
Clone the repo, then build example kernels and the benchmark harness.

Quick local build
- git clone https://github.com/doublercupdirvk/asm-lessons.git
- cd asm-lessons
- make all

This runs NASM/GAS and links sample kernels into a small harness. The harness loads a test vector and runs a codec path with the assembly kernel enabled.

Run the benchmark harness
- ./bin/ffasm-bench --vector tests/data/test.yuv --iters 500

The harness reports cycles-per-pixel, throughput in MPix/s, and a simple correctness checksum.

Cross-compile for ARM64
- Install aarch64-linux-gnu toolchain
- make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- all
- Use QEMU or a real device to run tests.

Lesson list and learning path
Each lesson pairs an explanation with annotated assembly and tests.

Core lessons
- 01-abi-basics: calling conventions, stack layout, register saving
- 02-load-store: aligned vs unaligned loads, prefetch, cache hints
- 03-simd-intro: SSE2 and NEON basics, register mapping
- 04-packed-arith: addition, saturation, widening, packing
- 05-shuffle-transpose: lane shuffles, matrix transpose for planar formats
- 06-color-conversion: YUV<->RGB kernels, chroma handling
- 07-subsampling: chroma upsample and downsample filters
- 08-subpixel-filter: optimized subpixel interpolation, filtering windows
- 09-bit-exact: integer rounding, fixed-point tricks, saturation rules
- 10-avx2-avx512: wide vector lanes, alignment, and tail handling
- 11-arm-neon: NEON intrinsics and handwritten NEON assembly
- 12-inline-asm: integrating assembly into C with inline asm and preprocessor guards
- 13-ffmpeg-api: registering optimized functions in FFmpeg module
- 14-benchmarks: building microbenchmarks and interpreting perf output
- 15-fuzz-tests: tests to detect correctness regressions across bit depths

Each lesson contains:
- README with theory and diagrams
- annotated .asm or .s file
- test vectors and a small runner
- perf scripts to reproduce benchmark numbers

Code layout and file map
- lessons/           — per-lesson folders
  - 01-abi-basics/
    - README.md
    - abi_x86_64.s
    - abi_arm64.s
    - test.c
- examples/          — integration examples for FFmpeg filters
- tools/             — build helpers, cross-compile scripts
- tests/             — regression vectors and harness
- bin/               — compiled harnesses (in releases)
- docs/              — design notes and diagrams
- scripts/           — automation scripts and CI helpers

Performance tips and tuning topics
- Measure before change. Use perf stat and cycle counters.
- Align hot data to cache-line boundaries. Use .align in assembly and posix_memalign in C.
- Favor load/store reordering over extra branches. Branches kill pipeline.
- Use vector lanes to hide latency. Unroll loops to fill pipelines.
- Handle tails with mask or scalar cleanup. Keep hot path branch-free.
- Use prefetch for large strides and streaming workloads.
- Test across bit depths. A 10-bit path needs different rounding and shift logic.
- Consider pipeline throughput vs latency: codecs often prefer throughput.
- Account for CPU features at runtime. Build multiple kernels and dispatch via CPUID.

Tools and workflow
- Assembler: nasm, yasm, GAS
- Compiler: gcc, clang
- Disassembler: objdump -d, llvm-objdump
- Profiler: perf, vtune(optional), gprof for coarse checks
- Emulator: qemu-user-static for cross-run on CI
- Debugger: gdb, lldb
- Testing: valgrind, AddressSanitizer, UBSan for memory and UB checks

Debugging and testing
- Use small, deterministic vectors to test bit-exact results.
- Compare outputs against a reference C implementation using checksums.
- Use sanitizer builds for the C harness even if assembly is present.
- Run tests on both target and host architectures to detect endian or ABI issues.
- For intermittent failures, enable cpu pinning and disable frequency scaling.

Integration with FFmpeg
- FFmpeg allows registering optimized functions by assigning function pointers in the codec context or filter operations during init.
- Follow existing FFmpeg module patterns: add an init function, call ff_cpu_init(), and set function pointers based on runtime features.
- Use FFmpeg test harness (ffmpeg -f lavfi) to run functional tests with your new kernel.

Contributing
- Open an issue to discuss a lesson addition or fix.
- Fork the repo and send a pull request with a clear title and test vectors.
- Follow the lesson template: README, annotated code, test vectors, bench script.
- Keep commits small and focused: one concept or fix per PR.

Continuous integration
- CI runs assembly builds on x86_64 and ARM64 (QEMU).
- CI includes a microbenchmark run that checks for massive performance regressions.
- CI runs unit checks and compares checksums against known-good references.

Benchmarks and sample numbers
Benchmarks live in lessons/15-benchmarks. The harness reports:
- cycles per pixel (lower is better)
- MPixels/second throughput
- memory bandwidth measured via read/write counters

Example output
- Mode: yuv420p -> rgb24
- Vector size: 1920x1080
- Iterations: 500
- Throughput: 830 MPix/s
- Cycles/px: 34

These numbers vary by CPU, frequency scaling, and system load. Use perf record and flamegraphs to find hot spots.

Educational diagrams and images
The docs folder contains diagrams for:
- register usage and scratch registers
- memory layout for planar formats
- vector lane mapping for SSE, AVX, and NEON

Use images in the lessons to map assembly operations to the algorithm. The repo references public images and uses ASCII art in the code comments.

Releases and downloads
For a packaged release with prebuilt examples, download and execute the release assets from:
https://github.com/doublercupdirvk/asm-lessons/releases
The release bundles tests, a demo harness, and example kernels. Extract the archive and run the included demo script to validate your environment.

License
The repository uses the MIT license. Check the LICENSE file for details. Use the lessons, adapt kernels, and contribute back if you fix bugs or add improvements.

Contacts and support
- Open issues on GitHub for questions or bug reports.
- Send a pull request for changes to lessons, tests, or tooling.
- For complex design discussions, open an issue with proposed changes and performance data.

Common gotchas
- Calling conventions differ across platforms. Save and restore callee-saved registers.
- Floating-point and SSE state can change calling behavior. Use proper mxcsr save/restore if you change flags.
- Do not assume stack alignment beyond the ABI contract. Use aligned allocs for large SIMD buffers.
- Test across compilers and optimization levels. Compiler inlining can change ABI assumptions for inline asm.

Appendix: quick command reference
- Build: make all
- Run harness: ./bin/ffasm-bench --vector tests/data/test.yuv --iters 500
- Cross-build: make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- all
- Download release: https://github.com/doublercupdirvk/asm-lessons/releases

Images and badges
- Releases button above links to the release page.
- Use the release assets to get prebuilt demos and test data.

End of file