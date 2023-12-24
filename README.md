# StringZilla 🦖

StringZilla is the Godzilla of string libraries, searching, splitting, sorting, and shuffling large textual datasets faster than you can say "Tokyo Tower" 😅

- ✅ Single-header pure C 99 implementation [docs](#quick-start-c-🛠️)
- Light-weight header-only C++ 11 `sz::string_view` and `sz::string` wrapper with the feature set of C++ 23 strings!
- ✅ [Direct CPython bindings](https://ashvardanian.com/posts/pybind11-cpython-tutorial/) with minimal call latency similar to the native `str` class, but with higher throughput [docs](#quick-start-python-🐍)
- ✅ [SWAR](https://en.wikipedia.org/wiki/SWAR) and [SIMD](https://en.wikipedia.org/wiki/Single_instruction,_multiple_data) acceleration on x86 (AVX2, AVX-512) and ARM (NEON, SVE)
- ✅ [Radix](https://en.wikipedia.org/wiki/Radix_sort)-like sorting faster than C++ `std::sort`
- ✅ [Memory-mapping](https://en.wikipedia.org/wiki/Memory-mapped_file) to work with larger-than-RAM datasets

Putting this into a table:

| Feature \ Library    |  STL | LibC |      StringZilla |
| :------------------- | ---: | ---: | ---------------: |
| Substring Search     |      |      |                  |
| Reverse Order Search |      |    ❌ |                  |
| Fuzzy Search         |    ❌ |    ❌ |                  |
| Edit Distance        |    ❌ |    ❌ |                  |
| Interface            |  C++ |    C | C , C++ , Python |


Who is this for?

- you want to process strings faster than default strings in Python, C, or C++
- you need fuzzy string matching functionality that default libraries don't provide
- you are student learning practical applications of SIMD and SWAR and how libraries like LibC are implemented
- you are implementing/maintaining a programming language or porting LibC to a new hardware architecture like a RISC-V fork and need a solid SWAR baseline

Limitations:

- Assumes little-endian architecture
- Assumes ASCII or UTF-8 encoding
- Assumes 64-bit address space

This library saved me tens of thousands of dollars pre-processing large datasets for machine learning, even on the scale of a single experiment.
So if you want to process the 6 Billion images from [LAION](https://laion.ai/blog/laion-5b/), or the 250 Billion web pages from the [CommonCrawl](https://commoncrawl.org/), or even just a few million lines of server logs, and haunted by Python's `open(...).readlines()` and `str().splitlines()` taking forever, this should help 😊

## Performance

StringZilla is built on a very simple heuristic:

> If the first 4 bytes of the string are the same, the strings are likely to be equal.
> Similarly, the first 4 bytes of the strings can be used to determine their relative order most of the time.

Thanks to that it can avoid scalar code processing one `char` at a time and use hyper-scalar code to achieve `memcpy` speeds.
__The implementation fits into a single C 99 header file__ and uses different SIMD flavors and SWAR on older platforms.

### Substring Search

| Backend \ Device         |                    IoT |                   Laptop |                    Server |
| :----------------------- | ---------------------: | -----------------------: | ------------------------: |
| __Speed Comparison__ 🐇   |                        |                          |                           |
| Python `for` loop        |                 4 MB/s |                  14 MB/s |                   11 MB/s |
| C++ `for` loop           |               520 MB/s |                 1.0 GB/s |                  900 MB/s |
| C++ `string.find`        |               560 MB/s |                 1.2 GB/s |                  1.3 GB/s |
| Scalar StringZilla       |                 2 GB/s |                 3.3 GB/s |                  3.5 GB/s |
| Hyper-Scalar StringZilla |           __4.3 GB/s__ |              __12 GB/s__ |             __12.1 GB/s__ |
| __Efficiency Metrics__ 📊 |                        |                          |                           |
| CPU Specs                | 8-core ARM, 0.5 W/core | 8-core Intel, 5.6 W/core | 22-core Intel, 6.3 W/core |
| Performance/Core         |         2.1 - 3.3 GB/s |              __11 GB/s__ |                 10.5 GB/s |
| Bytes/Joule              |           __4.2 GB/J__ |                   2 GB/J |                  1.6 GB/J |

### Split, Partition, Sort, and Shuffle

Coming soon.

## Quick Start: Python 🐍

1. Install via pip: `pip install stringzilla`  
2. Import the classes you need: `from stringzilla import Str, Strs, File`  

### Basic Usage

StringZilla offers two mostly interchangeable core classes:

```python
from stringzilla import Str, File

text_from_str = Str('some-string')
text_from_file = Str(File('some-file.txt'))
```

The `Str` is designed to replace long Python `str` strings and wrap our C-level API.
On the other hand, the `File` memory-maps a file from persistent memory without loading its copy into RAM.
The contents of that file would remain immutable, and the mapping can be shared by multiple Python processes simultaneously.
A standard dataset pre-processing use case would be to map a sizeable textual dataset like Common Crawl into memory, spawn child processes, and split the job between them.

### Basic Operations

- Length: `len(text) -> int`
- Indexing: `text[42] -> str`
- Slicing: `text[42:46] -> Str`
- String conversion: `str(text) -> str`
- Substring check: `'substring' in text -> bool`
- Hashing: `hash(text) -> int`

### Advanced Operations

- `text.contains('substring', start=0, end=9223372036854775807) -> bool`
- `text.find('substring', start=0, end=9223372036854775807) -> int`
- `text.count('substring', start=0, end=9223372036854775807, allowoverlap=False) -> int`
- `text.splitlines(keeplinebreaks=False, separator='\n') -> Strs`
- `text.split(separator=' ', maxsplit=9223372036854775807, keepseparator=False) -> Strs`

### Collection-Level Operations

Once split into a `Strs` object, you can sort, shuffle, and reorganize the slices.

```python
lines: Strs = text.split(separator='\n')
lines.sort()
lines.shuffle(seed=42)
```

Need copies?

```python
sorted_copy: Strs = lines.sorted()
shuffled_copy: Strs = lines.shuffled(seed=42)
```

Basic `list`-like operations are also supported:

```python
lines.append('Pythonic string')
lines.extend(shuffled_copy)
```

### Low-Level Python API

The StringZilla CPython bindings implement vector-call conventions for faster calls.

```py
import stringzilla as sz

contains: bool = sz.contains("haystack", "needle", start=0, end=9223372036854775807)
offset: int = sz.find("haystack", "needle", start=0, end=9223372036854775807)
count: int = sz.count("haystack", "needle", start=0, end=9223372036854775807, allowoverlap=False)
levenshtein: int = sz.levenshtein("needle", "nidl")
```

## Quick Start: C 🛠️

There is an ABI-stable C 99 interface, in case you have a database, an operating system, or a runtime you want to integrate with StringZilla.

```c
#include <stringzilla/stringzilla.h>

// Initialize your haystack and needle
sz_string_view_t haystack = {your_text, your_text_length};
sz_string_view_t needle = {your_subtext, your_subtext_length};

// Perform string-level operations
sz_size_t character_count = sz_count_char(haystack.start, haystack.length, "a");
sz_size_t substring_position = sz_find(haystack.start, haystack.length, needle.start, needle.length);

// Hash strings
sz_u32_t crc32 = sz_hash(haystack.start, haystack.length);

// Perform collection level operations
sz_sequence_t array = {your_order, your_count, your_get_start, your_get_length, your_handle};
sz_sort(&array, &your_config);
```

## Contributing 👾

Future development plans include:

- [x] [Replace PyBind11 with CPython](https://github.com/ashvardanian/StringZilla/issues/35), [blog](https://ashvardanian.com/posts/pybind11-cpython-tutorial/)
- [x] [Bindings for JavaScript](https://github.com/ashvardanian/StringZilla/issues/25)
- [ ] [Faster string sorting algorithm](https://github.com/ashvardanian/StringZilla/issues/45)
- [ ] [Reverse-order operations in Python](https://github.com/ashvardanian/StringZilla/issues/12)
- [ ] [Splitting with multiple separators at once](https://github.com/ashvardanian/StringZilla/issues/29)
- [ ] Splitting CSV rows into columns
- [ ] UTF-8 validation.
- [ ] Arm SVE backend
- [ ] Bindings for Java and Rust

Here's how to set up your dev environment and run some tests.

### Development

CPython:

```sh
# Clean up, install, and test!
rm -rf build && pip install -e . && pytest scripts/ -s -x

# Install without dependencies
pip install -e . --no-index --no-deps
```

NodeJS:

```sh
npm install && npm test
```

### Benchmarking

To benchmark on some custom file and pattern combinations:

```sh
python scripts/bench_substring.py --haystack_path "your file" --needle "your pattern"
```

To benchmark on synthetic data:

```sh
python scripts/bench_substring.py --haystack_pattern "abcd" --haystack_length 1e9 --needle "abce"
```

### Packaging

To validate packaging:

```sh
cibuildwheel --platform linux
```

### Compiling C++ Tests

Running benchmarks:

```sh
cmake -DCMAKE_BUILD_TYPE=Release -DSTRINGZILLA_BUILD_BENCHMARK=1 -B ./build_release
cmake --build build_release --config Release
./build_release/stringzilla_bench_substring
```

Comparing different hardware setups:

```sh
cmake -DCMAKE_BUILD_TYPE=Release -DSTRINGZILLA_BUILD_BENCHMARK=1 \
    -DCMAKE_CXX_FLAGS="-march=sandybridge" -DCMAKE_C_FLAGS="-march=sandybridge" \
    -B ./build_release/sandybridge && cmake --build build_release/sandybridge --config Release
cmake -DCMAKE_BUILD_TYPE=Release -DSTRINGZILLA_BUILD_BENCHMARK=1 \
    -DCMAKE_CXX_FLAGS="-march=haswell" -DCMAKE_C_FLAGS="-march=haswell" \
    -B ./build_release/haswell && cmake --build build_release/haswell --config Release
cmake -DCMAKE_BUILD_TYPE=Release -DSTRINGZILLA_BUILD_BENCHMARK=1 \
    -DCMAKE_CXX_FLAGS="-march=sapphirerapids" -DCMAKE_C_FLAGS="-march=sapphirerapids" \
    -B ./build_release/sapphirerapids && cmake --build build_release/sapphirerapids --config Release

./build_release/sandybridge/stringzilla_bench_substring
./build_release/haswell/stringzilla_bench_substring
./build_release/sapphirerapids/stringzilla_bench_substring
```

Running tests:

```sh
cmake -DCMAKE_BUILD_TYPE=Debug -DSTRINGZILLA_BUILD_TEST=1 -B ./build_debug
cmake --build build_debug --config Debug
./build_debug/stringzilla_test_substring
```

On MacOS it's recommended to use non-default toolchain:

```sh
# Install dependencies
brew install libomp llvm

# Compile and run tests
cmake -B ./build_release \
    -DCMAKE_C_COMPILER="gcc-12" \
    -DCMAKE_CXX_COMPILER="g++-12" \
    -DSTRINGZILLA_USE_OPENMP=1 \
    -DSTRINGZILLA_BUILD_TEST=1 \
    -DSTRINGZILLA_BUILD_BENCHMARK=1 \
    && \
    make -C ./build_release -j && ./build_release/stringzilla_bench_substring
```

## License 📜

Feel free to use the project under Apache 2.0 or the Three-clause BSD license at your preference.

---

If you like this project, you may also enjoy [USearch][usearch], [UCall][ucall], [UForm][uform], [UStore][ustore], [SimSIMD][simsimd], and [TenPack][tenpack] 🤗

[usearch]: https://github.com/unum-cloud/usearch
[ucall]: https://github.com/unum-cloud/ucall
[uform]: https://github.com/unum-cloud/uform
[ustore]: https://github.com/unum-cloud/ustore
[simsimd]: https://github.com/ashvardanian/simsimd
[tenpack]: https://github.com/ashvardanian/tenpack


# The weirdest interfaces of C++23 strings:

## Third `std::basic_string_view<CharT,Traits>::find`

constexpr size_type find( basic_string_view v, size_type pos = 0 ) const noexcept;
(1)	(since C++17)
constexpr size_type find( CharT ch, size_type pos = 0 ) const noexcept;
(2)	(since C++17)
constexpr size_type find( const CharT* s, size_type pos, size_type count ) const;
(3)	(since C++17)
constexpr size_type find( const CharT* s, size_type pos = 0 ) const;
(4)	(since C++17)


## HTML Parsing

```txt
<tag>       Isolated tag start
<tag\w      Tag start with attributes
<tag/>      Self-closing tag
</tag>      Tag end
```

In any case, the tag name is always followed by whitespace, `/` or `>`.
And is always preceded by whitespace. `/` or `<`.

Important distinctions between XML and HTML:

- XML does not truncate multiple white-spaces, while HTML does.