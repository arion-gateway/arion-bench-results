# arion-bench-results

Raw results, charts, and configs from [arion-bench](https://github.com/arion-gateway/arion-bench)
benchmark campaigns. Each top-level directory is one dated campaign.

## Layout

```
<campaign>/
├── README.md         campaign-specific notes: what was tested, caveats
├── proxy-configs/     the proxy/backend configs under test (arion-h1.yaml, nginx-h1.conf, ...)
├── sweep-configs/      arion-bench YAML configs -- one per (scenario, proxy)
└── results/
    └── <family>/
        ├── <timestamp>_<scenario>_<proxy>/results.csv
        └── comparison/<scenario>/*.png
```

`results/<family>/<timestamp>_<scenario>_<proxy>/` is what `arion-bench run`
produces (minus the per-run `run-NNN/` log directories, pruned here to keep
the repo small); `comparison/` is `arion-bench batch` output. A `<family>`
groups related scenarios so they're only ever compared against each other
(baseline vs. baseline, not baseline vs. ext_proc).

To regenerate charts from the raw CSVs:

```sh
pip install arion-bench   # or: pip install -e path/to/arion-bench
arion-bench batch --root <campaign>/results/<family>
```

## Campaigns

- [`202608_bench/`](202608_bench/README.md) -- H1/H2(+TLS) baselines, ext_proc,
  Cedar authz, MCP gateway, Wasm filters, across arion/envoy/nginx/haproxy/agentgateway.
