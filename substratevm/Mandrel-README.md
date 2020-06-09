# Substrate VM

## Introduction

This file describes differences between GraalVM CE and Mandrel distributions. See [README.md](./README.md) for inforamtion about Substrate VM.


## Quick start

For compilation `native-image` depends on the local toolchain, so make sure: `glibc-devel`, `zlib-devel` (header files for the C library and `zlib`) and `gcc` are available on your system. Mandrel distribution of `native-image` also needs `libffi-devel` available on your system.
