# 202608_bench

2026-08-27 to 2026-10-22. H1/H2(+TLS) baselines plus several feature
scenarios, across arion, envoy, nginx, haproxy, and agentgateway (not every
proxy in every scenario -- see each family below). The description of the
testbed is in [`analysis/testbed.md`](analysis/testbed.md).

| family | proxies | what it tests |
|---|---|---|
| `baseline` | arion, envoy, nginx, haproxy, agentgateway | H1, H1+TLS, H2+TLS->H1, H2+TLS->H2+TLS, 32 proxy cores |
| `baseline-small-scale` | arion, envoy, nginx, haproxy | same H1 baseline, 4 proxy cores |
| `cedar` | arion, nginx | Cedar authz policy evaluation on the request path |
| `ext_proc` | arion, envoy | ext_proc headers-only vs. headers+body |
| `mcp-forwarding` | arion, envoy, nginx, haproxy, agentgateway | plain L7 forwarding of MCP traffic, no MCP-aware filter |
| `mcp-gateway-{body,input-schema,path,path-body-query,query}` | arion, agentgateway | MCP gateway translating OpenAPI operations to MCP tool calls, one family per mapping strategy |
| `mcp-gateway-mcp-upstream[-sse]` | arion, agentgateway | MCP gateway in front of an MCP-native upstream (Streamable HTTP / SSE) |
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

## Analysis

In `analysis/` there is a more detailed analysis of some of the results:

- [`testbed.md`](analysis/testbed.md): the environment used for all the
  documents below.
- [`baseline.md`](analysis/baseline.md): the five proxies with the four
  H1/H2(+TLS) protocol combinations.
- [`baseline-small-scale.md`](analysis/baseline-small-scale.md): the same
  `h1_h1` scenario with only 4 cores for the proxy.
- [`mcp-forwarding.md`](analysis/mcp-forwarding.md) (also with the
  comparison against the HTTP baseline),
  [`mcp-gateway-mcp-upstream.md`](analysis/mcp-gateway-mcp-upstream.md) and
  [`mcp-gateway-input-schema.md`](analysis/mcp-gateway-input-schema.md): one
  document for each MCP scenario. The `mcp-gateway-{body,path,query,
  path-body-query}` variants do not have a dedicated document, because
  their results are very similar to `input-schema` (see that document).
