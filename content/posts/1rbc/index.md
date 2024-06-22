---
title: 1 Billion Rows Challenge
slug: 1rbc
description: 1 Billion Rows Challenge using all the tools in the toolbox
date: "2024-04-16"
tags: ["data-engineering", "rust", "python", "duckdb"]
subjects: ["python", "data-processing"]
showTaxonomies: true
---

# 1 Billion Rows Challenge

## Comparing Python and Rust for Processing Large Datasets

In this post, I will share my experience tackling the [1 Billion Rows Challenge](https://github.com/gunnarmorling/1brc). This consists in reading a dataset with 1 billion rows and 2 columns (city, temp) and computing the minimum, maximum, and mean temperaturefor each city. I approached this problem using both Python and Rust to compare their performance and capabilities. Below, I will walk you through the different strategies I employed for each language and the results of my benchmarks.

Note that all the code and benchmarks are available in the github repo:

{{< github repo="RomainEconomics/1rbc" >}}

To generate the file used for the computation, go the [1BRC repo](https://github.com/gunnarmorling/1brc).

## Benchmarks

As a teaser, here the results of the benchmarks:

| Strategy              | Language |   Mean | StdDev | Median |    Min |    Max |
| :-------------------- | :------- | -----: | -----: | -----: | -----: | -----: |
| v6_multi_threading_v1 | Rust     |   4.71 |   0.17 |   4.76 |   4.46 |   4.88 |
| baseline_ibis_duckdb  | Python   |  11.25 |   0.52 |  11.34 |  10.71 |  11.82 |
| baseline_polars       | Python   |  20.18 |  12.76 |  11.81 |  11.63 |  40.37 |
| v5_mmap               | Rust     |  29.85 |   1.12 |  29.34 |  29.27 |  31.84 |
| v5_pypy_mp_v2         | Python   |  34.77 |   9.42 |  32.78 |  24.41 |  46.29 |
| v4_parse_temp         | Rust     |  39.34 |   0.13 |  39.28 |  39.24 |  39.56 |
| v3_fast_float         | Rust     |  41.97 |   0.07 |  41.94 |  41.92 |  42.08 |
| v4_pypy_mp            | Python   |  42.67 |   7.67 |  39.17 |  34.52 |  54.08 |
| v2_faster_hash        | Rust     |  44.32 |   0.15 |  44.28 |  44.19 |   44.5 |
| v1_bytes              | Rust     |  53.73 |   0.78 |  54.14 |   52.4 |   54.3 |
| v0_buffer_reader      | Rust     |  77.43 |   5.76 |  75.23 |  74.19 |  87.68 |
| v3_pypy_temp_parsing  | Python   |  96.24 |   0.55 |  96.32 |  95.52 |  96.81 |
| v2_pypy_bytes         | Python   | 124.39 |   0.78 | 124.15 |  123.7 | 125.73 |
| v1_pypy_list          | Python   | 181.59 |   0.37 | 181.74 | 181.09 |    182 |
| v0_builtin_with_pypy  | Python   | 184.76 |   1.34 | 184.33 | 182.98 | 186.19 |
| baseline_pandas       | Python   | 217.59 |   4.66 | 215.74 | 214.61 | 225.84 |
| v0_builtin            | Python   | 703.82 |   12.6 | 699.51 | 695.57 | 725.97 |

## Python Strategies

### Baseline in Pure Python

I started with a baseline implementation in pure Python. This approach was straightforward but not optimized for performance.

```python
import sys

class TempStat:
    def __init__(self, temp: float):
        self.min_val = temp
        self.max_val = temp
        self.sum_val = temp
        self.count = 1

    def mean(self) -> float:
        return self.sum_val / self.count

    def update(self, temp: float):
        self.min_val = min(self.min_val, temp)
        self.max_val = max(self.max_val, temp)
        self.sum_val += temp
        self.count += 1


if __name__ == "__main__":
    file_path = sys.argv[1]

    cities: dict[str, TempStat] = {}

    with open(file_path, "r") as file:
        for idx, line in enumerate(file):
            city, temp = line.split(";")
            if city in cities:
                cities[city].update(float(temp))
            else:
                cities[city] = TempStat(float(temp))
```

This code runs, but is very slow (more than 700 seconds).

Several problems can be identified:

- It uses the python interpreter, which is not the most performant, as we will see.
- We read each line as a string, instead of bytes, which is slower.
- Each temperature is parsed to float
- Only one thread, and one core is used

We will try to improve on these points in the following sections.

But first, even if the goal is to use plain Python and Rust, as possible, it is interesting to compare our results to some more popular, and more optimized libraries.

### Using Common Libraries

Next, I leveraged some popular Python libraries known for their performance with large datasets:

- Pandas: A powerful data manipulation library. The most used one in Python, but not the fastest.
- Polars: A fast DataFrame library.
- DuckDB: An in-process SQL OLAP database management system.

As an example, the code for DuckDB (using Ibis as a wrapper around it):

```python
import sys
import ibis

if __name__ == "__main__":
    file_path = sys.argv[1]

    data = ibis.read_csv(file_path, names=["city", "temp"], sep=";")

    res = (
        data.group_by("city")
        .aggregate(
            min_val=data.temp.min(),
            max_val=data.temp.max(),
            mean_val=data.temp.mean().round(1),
        )
        .order_by("city")
    )

    df = res.to_pandas()
```

The advantage of using those libraries, is that it can be really easy to use, and they are already optimized for performance (the code runs in almost 10 sec on my machine).
Except for Pandas, for which, if you try to read the file with the default parameters, you will likely run out of memory.

### PyPy

I also tried running the code with PyPy, a just-in-time compiler for Python, to see if it could offer any performance improvements.

We went from 700 seconds to 184 seconds, which is a good improvement without changing any code. We will thus stick with that interpreter for the following optimizations.

### Optimizations

- [Reading File Bytes](https://github.com/RomainEconomics/1rbc/blob/master/python/v2_pypy_bytes.py): Instead of reading the file line by line, I read the entire file as bytes.
  - 60 seconds speedup
- [Parsing Temperature as Integer](https://github.com/RomainEconomics/1rbc/blob/master/python/v3_pypy_temp_parsing.py): Parsing the temperature as an integer instead of a float to save processing time.
  - Almost 30 seconds gained, from 124 to 96 seconds
- [Multiprocessing](https://github.com/RomainEconomics/1rbc/blob/master/python/v5_pypy_mp_v2.py): Since Python's Global Interpreter Lock (GIL) limits the effectiveness of multithreading, I used multiprocessing to parallelize the task.
  - 62 seconds - from 96 to 34 seconds

Reading the file as bytes and parsing the temperature as an integer were the most significant optimizations before we were able to use multiprocessing.

However, to read a file and process it using multiple core, we need to first ensures each core sees different chunks.

For that, I defined this function:

```python
def find_chunk_boundaries(filename: str, workers: int) -> list[tuple[int, int]]:
    file_size = os.path.getsize(filename)
    chunk_size = file_size // workers
    chunks = []

    def find_new_line(f: io.BufferedReader, start: int):
        f.seek(start)
        while True:
            chunk = f.read(2048)
            if b"\n" in chunk:
                return start + chunk.index(b"\n") + 1
            if len(chunk) < 2048:
                return f.tell()
            start += len(chunk)

    with open(filename, "rb") as f:
        start = 0
        for _ in range(workers):
            end = find_new_line(f, start + chunk_size)
            chunks.append((start, end))
            start = end
    return chunks
```

This function will return the boundaries of the chunks that each worker will process.
Moreover, we need to ensure that the end of a chunk fits exactly the end of a line, to avoid splitting a line between two workers.

Using all the cores on my machine (20) allowed to divide by almost 3 the time needed to process the file.

Coming from 700 seconds, we are now at 34 seconds, which is a good improvement.

But we're still far from the performance of Polars or DuckDB.

## Rust Strategies

### Baseline in Pure Rust

I started with a baseline implementation in pure Rust using a [buffered reader](https://github.com/RomainEconomics/1rbc/blob/master/rust/src/bin/v0_buffer_reader.rs).

### Optimizations

- [Using Bytes](https://github.com/RomainEconomics/1rbc/blob/master/rust/src/bin/v1_bytes.rs): Reading the file as bytes for faster processing.
- [Faster Hash Map](https://github.com/RomainEconomics/1rbc/blob/master/rust/src/bin/v2_faster_hash.rs): Using a more efficient hash map for storing city temperatures.
- [Faster Float Parsing](https://github.com/RomainEconomics/1rbc/blob/master/rust/src/bin/v3_fast_float.rs): Optimizing the parsing of temperature values.
- [Parsing Temperature as Integer](https://github.com/RomainEconomics/1rbc/blob/master/rust/src/bin/v4_parse_temp.rs): Similar to the Python approach, parsing the temperature as an integer.
- [Memory-Mapped Files (mmap)](https://github.com/RomainEconomics/1rbc/blob/master/rust/src/bin/v5_mmap.rs): Using memory-mapped files for faster file I/O.
- [Multithreading](https://github.com/RomainEconomics/1rbc/blob/master/rust/src/bin/v6_multi_threading_v1.rs): Utilizing Rust's powerful multithreading capabilities with Arc (Atomic Reference Counting) for shared state.

## Results

| Strategy              | Language |   Mean | StdDev | Median |    Min |    Max |
| :-------------------- | :------- | -----: | -----: | -----: | -----: | -----: |
| v6_multi_threading_v1 | Rust     |   4.71 |   0.17 |   4.76 |   4.46 |   4.88 |
| baseline_ibis_duckdb  | Python   |  11.25 |   0.52 |  11.34 |  10.71 |  11.82 |
| baseline_polars       | Python   |  20.18 |  12.76 |  11.81 |  11.63 |  40.37 |
| v5_mmap               | Rust     |  29.85 |   1.12 |  29.34 |  29.27 |  31.84 |
| v5_pypy_mp_v2         | Python   |  34.77 |   9.42 |  32.78 |  24.41 |  46.29 |
| v4_parse_temp         | Rust     |  39.34 |   0.13 |  39.28 |  39.24 |  39.56 |
| v3_fast_float         | Rust     |  41.97 |   0.07 |  41.94 |  41.92 |  42.08 |
| v4_pypy_mp            | Python   |  42.67 |   7.67 |  39.17 |  34.52 |  54.08 |
| v2_faster_hash        | Rust     |  44.32 |   0.15 |  44.28 |  44.19 |   44.5 |
| v1_bytes              | Rust     |  53.73 |   0.78 |  54.14 |   52.4 |   54.3 |
| v0_buffer_reader      | Rust     |  77.43 |   5.76 |  75.23 |  74.19 |  87.68 |
| v3_pypy_temp_parsing  | Python   |  96.24 |   0.55 |  96.32 |  95.52 |  96.81 |
| v2_pypy_bytes         | Python   | 124.39 |   0.78 | 124.15 |  123.7 | 125.73 |
| v1_pypy_list          | Python   | 181.59 |   0.37 | 181.74 | 181.09 |    182 |
| v0_builtin_with_pypy  | Python   | 184.76 |   1.34 | 184.33 | 182.98 | 186.19 |
| baseline_pandas       | Python   | 217.59 |   4.66 | 215.74 | 214.61 | 225.84 |
| v0_builtin            | Python   | 703.82 |   12.6 | 699.51 | 695.57 | 725.97 |

## Analysis

### Rust

Rust consistently outperformed Python in all benchmarks for single threaded code. The fastest Rust implementation, which used multithreading, completed the task in just 4.71 seconds on average.

### Python

Python, while slower than Rust, offers a rich ecosystem of libraries that can call C, C++, or even Rust code, making it easier to write and maintain. The fastest Python implementation using DuckDB completed the task in 11.25 seconds on
average. PyPy provided some performance improvements, but it was still not as fast as Rust.

## Conclusion

This exercise was a valuable learning experience. It highlighted the performance differences between Python and Rust and provided insights into various optimization techniques. While Rust is clearly faster, Python's ease of use and extensive library support make it a strong contender for many applications.

Notably, I learned a lot about using multithreading in Rust (without using Rayon, a common library used for multithreading), memory-mapped files, and other optimization strategies. If performance is critical and you are comfortable with Rust, this seems like a good choice. However, for ease of development and leveraging existing libraries, Python remains a powerful tool.
