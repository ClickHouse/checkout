# A thin wrapper for actions/checkout

## Why?

A few reasons:

- Separate control for submodules' depth
- The submodules' checkout time drastically decreased from 5 minutes to 1:20 for ClickHouse/ClickHouse CI
- Added a very common step to clean out the old code
- `--jobs` for fetching submodules surprisingly does much less parallel work under the hood than possible, see [comparison](#submodules-checkout-difference) of different time for different methods.

### Submodules checkout difference
Here's the difference between native git submodules' checkout difference and a custom one:

```
# Prepare repo, launched before each measurment
$ rm -rf ClickHouse/ && time git clone git@github.com:ClickHouse/ClickHouse.git --no-tags --progress --no-recurse-submodules --depth=1 --branch=v23.1.1.3077-stable
...
real	0m9.553s
user	0m4.253s
sys	0m1.359s

$ time (
  git -C ClickHouse submodule sync && \
  git -C ClickHouse submodule init && \
  git config --file ClickHouse/.gitmodules --null --get-regexp path | \
    sed -z 's|.*\n||' | \
    xargs --max-procs=100 --null --no-run-if-empty --max-args=1 \
      git -C ClickHouse submodule update --depth=1 --single-branch
)
...
real	1m48.657s
user	1m34.088s
sys	0m31.364s

$ time (
  git -C ClickHouse submodule sync && \
  git -C ClickHouse submodule init && \
  git -C ClickHouse submodule update --depth=1 --single-branch --jobs=100
)
...
real	2m48.604s
user	1m29.671s
sys	0m30.433s

$ time (
  git -C ClickHouse submodule sync && \
  git -C ClickHouse submodule init && \
  git -C ClickHouse submodule update --depth=1 --single-branch
)
...
real	6m16.782s
user	1m29.483s
sys	0m29.985s

$ time (
  git -C ClickHouse submodule sync && \
  git -C ClickHouse submodule init && \
  git -C ClickHouse submodule update --jobs=100
)
...
real	11m10.315s
user	16m56.844s
sys	1m43.773s

$ time (
  git -C ClickHouse submodule sync && \
  git -C ClickHouse submodule init && \
  git -C ClickHouse submodule update
)
...
real	12m57.784s
user	16m37.995s
sys	1m30.242s
```

## Parameters

- `clear-repository`: if the repository's directory should be deleted before cloning
- `submodules-depth`: a separate setting for submodule's checkout

The parameters for the upstream `actions/checkout`

- `fetch-depth`
- `ref`
- `submodules`
- `token`
