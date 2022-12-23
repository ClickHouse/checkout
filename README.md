# A thin wrapper for actions/checkout

## Why?

A few reasons:

- Separate control for submodules' depth
- The submodules' checkout time drastically decreased from 5 minutes to 1:20 for ClickHouse/ClickHouse CI
- Added a very common step to clean out the old code

## Parameters

- `clear-repository`: if the repository's directory should be deleted before cloning
- `submodules-depth`: a separate setting for submodule's checkout

The parameters for the upstream `actions/checkout`

- `fetch-depth`
- `ref`
- `submodules`
- `token`
