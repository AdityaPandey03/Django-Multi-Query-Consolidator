

# Django Multi-Query Consolidator

Executes multiple querysets over a single db query and returns results which would have been returned in normal evaluation of querysets. This can help in critical paths to avoid network/connection-pooler latency if db cpu/mem are nowhere near exhaustion. Ideal use case is fetching multiple small and optimised independent querysets where above mentioned latencies can dominate total execution time.

Supports only Postgres as of now

## Installation

```bash
pip install django-querysets-single-query-fetch
```

## Usage

```py

from django_querysets_single_query_fetch.service import QuerysetsSingleQueryFetch

querysets = [queryset1, queryset2, ...]
results = QuerysetsSingleQueryFetch(querysets=querysets).execute()

assert results == [list(queryset) for queryset in querysets]
```

Following tests pass (assuming no `prefetch_related` in querysets)

```py

# without (no. of queries is equal to no. of querysets)
with self.assertNumQueries(len(querysets)):
    results = [list(queryset) for queryset in querysets]

# with (irrespective of no. of querysets, only one network call is made)
with self.assertNumQueries(1):
    results = QuerysetsSingleQueryFetch(querysets=querysets).execute()

```

Fetching count of queryset using `QuerysetCountWrapper` (since `queryset.count()` is not a lazy method)

```py
from django_querysets_single_query_fetch.service import QuerysetsSingleQueryFetch, QuerysetCountWrapper

querysets = [QuerysetCountWrapper(queryset=queryset1), queryset2, ...]
results =  QuerysetsSingleQueryFetch(querysets=querysets) 

assert results == [queryset1.count(), list(queryset2), ...] 
```
