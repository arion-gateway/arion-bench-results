# 202608_bench

2026-08-27 to 2026-09-21. H1/H2(+TLS) baselines plus several feature
scenarios, across arion, envoy, nginx, haproxy, and agentgateway (not every
proxy in every scenario -- see each family below).

| family | proxies | what it tests |
|---|---|---|
| `baseline` | arion, envoy, nginx, haproxy | H1, H1+TLS, H2+TLS->H1, H2+TLS->H2+TLS, full core count |
| `baseline-small-scale` | arion, envoy, nginx, haproxy | same H1 baseline, narrow core pinning |
| `cedar` | arion, nginx | Cedar authz policy evaluation on the request path |
| `ext_proc` | arion, envoy | ext_proc headers-only vs. headers+body |
| `mcp-gateway-{body,input-schema,mcp-upstream,path,path-body-query,query}` | arion, agentgateway | MCP tool-call gateway, one family per OpenAPI-to-MCP mapping strategy |
| `wasm-benchmark` | arion, envoy, nginx | Wasm filter chain, generic benchmark filter |
| `wasm-header-map` | arion, envoy, nginx | Wasm filter chain, header-map-heavy filter |

## Layout

- `proxy-configs/` -- the proxy and backend configs under test, flat,
  `<proxy>-<protocol>[-<variant>].<ext>`. A few absolute paths (Wasm filter
  `.wasm` binaries, agentgateway's OpenAPI spec files) have been replaced with
  `/path/to/...` placeholders -- point them at your own checkout to rerun.
- `sweep-configs/` -- the matching `arion-bench` YAML config per
  `<scenario>_<proxy>`, e.g. `h1_h1_arion.yaml` drives
  `results/baseline/*_h1_h1_arion/`. Proxy binary paths are `/path/to/...`
  placeholders (point at your own build); the `-c`/`-f proxy-configs/...`
  argument is a real relative path, so run `arion-bench` from inside this
  directory (`202608_bench/`), e.g.
  `arion-bench run sweep-configs/h1_h1_arion.yaml`.
- `results/<family>/` -- see the top-level README for the shape.

One naming quirk: `h1_h1_small_scale_<proxy>.yaml` drives runs saved under
`results/baseline-small-scale/*_h1_h1_<proxy>/` -- the scenario there is
`h1_h1`, not `h1_h1_small_scale`; the `baseline-small-scale` family name
already carries that distinction, so the run directories don't repeat it.

## Caveats

These scenarios were originally run from one-off hand-edited scripts, later
converted into the `sweep-configs/` checked in here. The conversion is a
*cleanup*, not a transcription -- a few things it changed relative to what
literally produced the bundled `results.csv` files:

- the generator's core pinning was widened to `0-62:2` (except small-scale
  runs, left as originally pinned);
- the nginx binary path was normalized to one build, where some historical
  runs used the system `nginx`;
- `--client compio` was dropped from the generator -- it had been used
  inconsistently (not the same proxies at the same rpc values within one
  scenario), and it wins only at `--rpc` unlimited while losing badly at low
  `--rpc`, so some historical `rpc=unlimited` numbers in these CSVs may run
  faster than a replay from `sweep-configs/` would.
