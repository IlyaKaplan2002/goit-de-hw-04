# Spark Jobs Analysis: 5 Jobs, 8 Jobs, and 7 Jobs

This document provides an analysis of how Spark processes jobs under different scenarios. By modifying the code execution flow with actions like `collect()` and `cache()`, we can observe the impact on the number of jobs executed by Spark.

---

## Scenario 1: **5 Jobs**

### What Happens:

The code performs the following steps:

1. Loading a CSV file (`spark.read.csv()`).
2. Repartitioning the data (`repartition`).
3. Filtering the data (`where`).
4. Grouping the data (`groupBy` and `count`).
5. Filtering grouped results (`where("count > 2")`).
6. Collecting the results with `collect()`.

### Explanation:

Spark executes transformations lazily, meaning that the pipeline is only triggered when an action (like `collect()`) is called. In this scenario, all transformations are optimized and executed in a single flow, resulting in **5 Jobs**:

- Reading the data.
- Repartitioning.
- Filtering.
- Grouping.
- Collecting the results.

---

## Scenario 2: **8 Jobs**

### What Happens:

1. The first action, `nuek_processed.collect()`, is executed.
2. A new transformation (`where("count > 2")`) is added.
3. A second action, `collect()`, is called.

### Explanation:

The first `collect()` triggers Spark to execute all prior transformations (`repartition`, `where`, `groupBy`, `count`) in **5 Jobs**. After this:

- The new transformation (`where("count > 2")`) and the second `collect()` action trigger **3 additional Jobs**:
  - Reading the cached or intermediate data.
  - Applying the filter.
  - Collecting the results.

### Total Jobs:

- **5 Jobs** for the first `collect()`.
- **3 Jobs** for the additional operations.

**Overall Total: 8 Jobs**

---

## Scenario 3: **7 Jobs**

### What Happens:

1. The `cache()` method is used before the first `collect()` call.
2. The first `collect()` action executes and caches the results.
3. The second transformation (`where("count > 2")`) is applied to the cached data.
4. A second `collect()` action is executed.

### Explanation:

Using `cache()` saves intermediate results in memory, reducing redundant computations. The workflow breaks down as follows:

- **4 Jobs** are required to compute and cache the initial data.
- **3 Jobs** are executed to filter the cached data and perform the second `collect()`.

### Total Jobs:

- **4 Jobs** for caching.
- **3 Jobs** for subsequent operations.

**Overall Total: 7 Jobs**

---

## Key Insights:

1. **Lazy Evaluation**: Spark only triggers transformations when an action (like `collect()`) is called.
2. **Intermediate Actions**: Adding intermediate actions like `collect()` breaks the execution flow, creating additional jobs for subsequent operations.
3. **Caching**: Using `cache()` optimizes workflows by avoiding redundant computations, reducing the total number of jobs executed.

By understanding how Spark processes transformations and actions, developers can optimize their workflows for better performance and efficiency.
