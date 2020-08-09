---
layout: post
title: Robust Pandas pipelines
category: Practical
categories: ['Python','data wrangling', 'pipelines', 'reproducible']
tags: ['Python','data wrangling', 'pipelines', 'reproducible']
---

---
<h2 class="no_toc">Table of Contents</h2>

* TOC
{:toc}

Recently I built a data pipeline in Pandas that would be run on a weekly basis.

The pipeline begins with querying one of our databases and ends with a csv that would be uploaded into another database.

You would think that this is trivial and things would just work.

Well, they don't unfortunately and so in the process I added multiple checks for potential errors which made my QA easier and ultimately the pipeline much more robust.

Today I've compiled a short list of ideas and code snippets that I found useful and will be incorporating into future data processing pipelines.

Note: The code here is for `Python3.6+`

## Basic checks for robustness

We should always check the following items at appropriate checkpoints.

__1) Columns__

```python
list(df.columns) == list(df2.columns)
```

We can explicitly specify the column names as well. The preferred approach is to use a dictionary.

```python
df = df.rename(
    columns={
        'a': 'customer_id',
        'b': 'salary',
        'c': 'age',
    }
)
```

__2) Data types__

When comparing data types, make sure the columns are in the same order.

```python

columns = ['customer_id', 'salaray', 'age']

list(df[columns].dtypes) == list(df2[columns].dtypes)
```

For more robustness, we can explicitly convert the features to specific data types with

```python
df = df.astype(
    dtype={
        'customer_id': str,
        'salary': float,
        'age': int,
    }
)
```

__3) Duplicate values__

This one varies depending on our use case. Suppose we have customer ID and this is supposed to be unique, then we want to make sure that this is the case with

```python
df.shape[0] == df['customer_id'].nunique()
```
Checking for nulls is a similar story.
```python
df.shape[0] == df['customer_id'].isnull().value_counts()[0]
```

__4) Grouping features__
When working with multiple DataFrames with the intention of joining them together, it's best to ensure that they are consistent.

The approach I am currently using is to keep a dictionary of various components of a DataFrame.

That is,
```python

cat_cols = [
    'country',
    'industry',
    'status',
    'education'
]
num_cols = [
    'salary',
    'age',
    'points'
]

target_var = [
    "credit_risk"
]

features = cat_cols + num_cols
training = features + target_var

# Create a model...
model.fit(df[training])
model.predict(df[features])
```
We separate the different types of variables for clarity and input into different processing pipelines.

If we have model configurations, it's good to put them in a dictionary as well for clarity.

__5) Chaining multiple functions together__

This is a minor one but for readability, it's better to style our Pandas DataFrame transformations as follows.
```python
df.groupby(['industry'])['age'] \
    .mean() \
    .reset_index() \
    .rename(columns={
        'industry': 'job_type'
        'age': 'avg_age',
    }) \
    .sort_values('avg_age', ascending=True)
```

__6) Saving to csv__

As you might have guessed, I will be specifying columns whenever I want to save a DataFrame to a csv files.
```python
df[columns_to_keep].to_csv(path, index=False)
```

__7) Viewing duplicates__

Normally we wouldn't want to drop duplicates without checking it, and suppose we want to compare rows where certain features are duplicated. Then this snippet returns all duplicates specified by the columns `['customer_id', 'salary']`.
```
df[df.duplicated(['customer_id', 'salary'], keep=False)]
```

__8) Avoiding the problem__

One could argue that by being explicit on the columns, we are avoiding the problem of unwanted columns.

Well in this case we simply do a check

```python
if set(columns_to_keep) == set(df.columns):
    raise ValueError("Different columns between 'columns to keep' and df.columns!")
```
and if this is not equal throw an error.

## Working with multiple DataFrames

__1) Concatenating/appending DataFrames__

If we are working with multiple DataFrames or partitioned data, or trying to combine previous data with new data, it is best to double-check that they actually have identical data types.

Suppose we have an array of DataFrames. Then list comprehension makes our life easy.
```python
columns_to_keep = ['customer_id', 'salaray', 'age']

columns_dict = {
    'a': 'customer_id',
    'b': 'salary',
    'b': 'age',
}
type_dict = {
    'customer_id': str,
    'salary': float,
    'age': int,
}
df_array = [df1, ..., df100]

df_array_processed = [
    df.rename(columns=columns_dict) \
        .astype(dtypes=type_dict) \
    [columns_to_keep]
    for df in df_array
]

df_all = pd.concat(df_array_processed, ignore_index=True)
```
Here we make sure that the columns, column names and types are identical. We can also set `ignore_index=True` to reset the index as well.
<!-- ```python
pd.concat([df[columns_to_keep] for df in df_array_processed])
``` -->

The reason why we want to maintain the column order is because if we are doing `pd.concat()` or `DataFrame.append()` it will generate a warning (older version of Pandas)
```bash
FutureWarning: Sorting because non-concatenation axis is not aligned. A future version of pandas will change to not sort by default. To accept the future behavior, pass 'sort=True'. To retain the current behavior and silence the warning, pass sort=False.
```
or it might fail.

This occurs when our axis's are not identical or poorly aligned.

The best approach is to be explicit and ensure that both our DataFrames have identical columns.

__2) Joining DataFrames__

At the end of the day, `df.join()` and `df.merge()` serve the same purpose, but `join()` joins to the other DataFrame using `df`'s index.

We can specify a column to join on for the other DataFrame.

This is less flexible but faster.

However, there are some caveats to this.

Whenever we filter or subset a DataFrame, the index remains unchanged and hence the index will not go from `0` to `n-1` as it would on the original DataFrame.

So it is important to `reset_index()` before joining to avoid running into issues.

For example
```python
df1 = pd.DataFrame({
  'a': ['1', '3', '2', '', '4', '5', ''],
  'col': ['a', 'b', 'c', 'd', 'e', 'f', 'g']
})
df2 = pd.DataFrame({
  'a': ['2', '3', '1', '', '6', '7', ''],
  'b': ['2', '3', '1', 'hello', '6', '7', 'world'],
  'col': ['a', 'b', 'c', 'd', 'e', 'f', 'g']
})

df3 = df2.set_index(pd.Index([i for i in range(5, df2.shape[0] + 5)]))

df1.join(df3, how='outer', lsuffix='_left', rsuffix='_right')
```
will not give us the result we want.

Note that we manually set the index here but if we're filtering or doing anything that changes the index, then `df.join()` might not output the desired result.

__3) Merging DataFrames__

The big one I picked up was merging on strings.

If any of the strings are empty on either side, then it will be a cross product - this is standard behaviour for most joins (SQL or otherwise) but easy to forget.

```python
df1 = pd.DataFrame({
  'a': ['1', '3', '2', '', '4', '5', ''],
  'col': ['a', 'b', 'c', 'd', 'e', 'f', 'g']
})
df2 = pd.DataFrame({
  'a': ['2', '3', '1', '', '6', '7', ''],
  'b': ['2', '3', '1', 'hello', '6', '7', 'world'],
  'col': ['a', 'b', 'c', 'd', 'e', 'f', 'g']
})

# Uncomment this and try again
# df1['a'] = df1['a'].replace('', np.nan)

df1.merge(
    df2,
    on='a',
    how='outer',
    suffixes=('_left', '_right')
)
```
The results are different, and hence empty strings are also worth checking for.
```python
df[df['a'] == '']
```

__4) Sanity Checks__

After we `merge`, `join` or `concat` please check the DataFrame shapes!

__5) Saving Empty DataFrames__

Sometimes we want to save an empty DataFrame. We can do exactly the same thing with

```python
empty_df = pd.DataFrame(columns=columns_to_keep).to_csv(path, index=False)
```
This is really useful if we want to wipe data that is incorrect but want to keep the columns so we don't run into issues when merging DataFrames together.

Also, it avoids having to do checks with `os.path.exists` and we know that this file is always meant to be there. It just doesn't add any new data when we're concatenating.

## A subtlety

Suppose we want to compare equality on two DataFrames for all values.

```python
df1 = pd.DataFrame({'a': [1, 2, 3, 1, 5], 'b': [6, 7, 8, 9, 10]})
df2 = pd.DataFrame({'a': [1, 2, 3, 44, 5], 'b': [6, 7, 8, 9, 10]})

all(df1 == df2)
```
Clearly these two DataFrames have different values right?

Try running the code and see what happens. :)

__Can you figure out why?__

## Conclusion

Data science or not, writing good code and anticipating potential errors with sanity checks and tests will improve our output and make our life easier.

Please let me know if you have any more tips and tricks when it comes to writing robust Pandas pipelines! :D