# Madras Python

Python bindings for reading (and writing) Madras Sorcery `.mdsi` files, built with pybind11 directly on top of the C++ reader/writer, with pandas and PyArrow-friendly output.

## Madras Sorcery

Madras Sorcery is a compact, static datastore where a single `.mdsi` file is simultaneously a compressed column store *and* a sorted, directly-navigable index — no separate index file, no decompress-then-scan step. See the [madras_sorcery](https://github.com/siara-in/madras_sorcery) super-repo for the full project overview.

## Getting started

Requires `pybind11`, `numpy`, and a C++11 compiler. Expects [madras_sorcery_core](https://github.com/siara-in/madras_sorcery_core)'s `include/` directory at `../include` relative to this repository — clone it as a sibling, or adjust `include_dirs` in `setup.py`.

```bash
pip install pybind11 numpy
pip install -e .
```

Reading a file:

```python
from madras import MadrasReader

r = MadrasReader("babynames.db.mdsi")
r.metadata                 # dict: rows, pk_columns, columns[]
r.to_pandas()               # full table -> pandas.DataFrame
r.to_arrow()                 # full table -> pyarrow.Table
r.lookup("name", "John")      # index/word lookup -> pandas.DataFrame of matches

for batch in r.batches(batch_size=200_000):
    ...  # pyarrow.RecordBatch, for streaming large sources
```

Writing a file:

```python
from madras import copy_from

copy_from(df, "out.mdsi", pk_columns=["state", "name"], word_index=["bio"])
```

These bindings don't go through Spark. For PySpark, use [madras_java](https://github.com/siara-in/madras_java#getting-started)'s Data Source V2 connector directly (`spark.read.format("madras").load(...)`, with its JAR on the classpath) rather than this package — or, if you just need a one-off export, `r.to_arrow().to_pandas()` here and write Parquet for Spark to read.

## License

This work is licensed under the MIT License. See [LICENSE](LICENSE).

## Support

Please feel free to communicate suggestions, improvements and corrections by creating issues here or send email to Arundale Ramanathan at arun@siara.in.
