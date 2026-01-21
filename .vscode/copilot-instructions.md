# Copilot Instructions for Module 1: Tabular Data

When writing R code for this module, please follow these guidelines for high-performance data handling:

## Using `duckdbfs` for Data Access
Always prefer `duckdbfs::open_dataset()` over `read.csv()` or `read_parquet()` for opening local or remote tabular datasets. This creates a lazy connection that doesn't load everything into memory.

```r
library(duckdbfs)
library(dplyr)

# Examples of opening datasets (can be URLs or local paths)
url <- "https://github.com/duckdb/duckdb/raw/main/data/parquet-testing/hive-partitioning/union_by_name/x=1/f1.parquet"
ds <- open_dataset(url)
```

## Data Manipulation with `dplyr`
Use standard `dplyr` verbs (`filter`, `select`, `mutate`, `group_by`, `summarise`) to process data. These operations are translated into optimized SQL by `dbplyr` and executed within DuckDB.

## Mindful Memory Management
Avoid `collect()` unless specifically necessary to bring the final, aggregated result into R memory for plotting or specialized R-only functions.

- **CAUTION**: Do not use `collect()` on large datasets without first filtering or summarizing to reduce the size.
- **TIP**: If you need to save a large intermediate result that exceeds RAM, use `write_dataset()` instead of `collect()`.

## Examples

### 1. Filtering and Summarizing Remotely
```r
# This processes data efficiently on the remote/disk without loading full rows
result <- ds |>
  filter(i > 10) |>
  group_by(j) |>
  summarise(mean_x = mean(x, na.rm = TRUE)) |>
  collect()
```

### 2. Handling Larger-than-RAM Data
Instead of collecting and then saving, write directly to parquet.
```r
# Efficiently streams data from source to a partitioned local directory
ds |>
  filter(condition) |>
  group_by(partition_col) |>
  write_dataset("local_archive")
```

### 3. Spatial Operations (Advanced)
DuckDB supports spatial operations for later modules. Keep operations in DuckDB as much as possible.
