# Copilot Instructions for Module 1: Tabular Data

When writing R code for this module, please follow these guidelines for high-performance data handling:

## Using `duckdbfs` for Data Access
Always prefer `duckdbfs::open_dataset()` over `read.csv()` or `read_parquet()` for opening local or remote tabular datasets. This creates a lazy connection that doesn't load everything into memory.

### S3 Configuration (Public Access)
To access public S3 buckets (like those on Source Cooperative), configure `duckdb_secrets` with empty credentials to avoid conflicts with local environment variables:

```r
library(duckdbfs)
library(dplyr)

duckdb_secrets(
    key = "",
    secret = "",
    endpoint = "s3.amazonaws.com",
    region = "us-west-2"
)

# Use the recursive ** pattern for partitioned datasets
s3_url <- "s3://us-west-2.opendata.source.coop/youssef-harby/exiobase-3/4588235/parquet/**"
ds <- open_dataset(s3_url)
```


## Data Manipulation with `dplyr` & Tidyverse
Follow modern tidyverse conventions:
- Use the native pipe `|>` instead of `%>%`.
- Use descriptive variable names (snake_case).
- Leverage standard `dplyr` verbs (`filter`, `select`, `mutate`, `group_by`, `summarise`). These operations are translated into optimized SQL by `dbplyr` and executed within DuckDB.

## Plotting & Visualization
When visualizing data:
- Always `collect()` the data into R memory before passing it to `ggplot2`.
- Use `ggplot2` for all plots.
- Ensure the data being collected is appropriately filtered or summarized to fit in memory.

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

### 2. Plotting with ggplot2
```r
library(ggplot2)

ds |>
  filter(year == 2022) |>
  group_by(region) |>
  summarize(total_value = sum(x, na.rm = TRUE)) |>
  collect() |> # Essential for ggplot2
  ggplot(aes(x = region, y = total_value)) +
  geom_col() +
  theme_minimal()
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
