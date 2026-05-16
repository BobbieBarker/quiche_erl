# quiche_erl

QUIC protocol bindings for Erlang via Cloudflare quiche. Hand-rolled C NIF wrapping `libquiche.a`.

## Stack

- Erlang/OTP 27+, rebar3 build system
- C11 NIF layer linking `libquiche.a` (Cloudflare quiche with `--features ffi`)
- quiche bundles BoringSSL (first build needs cmake + Go)
- Test: Common Test + PropEr
- Published to Hex as an Erlang package

## Architecture

Two-layer design:

- `quiche_erl` (facade): public API, option normalization, async message handling
- `quiche_erl_nif` (internal): NIF stubs, `erlang:load_nif/2`, resource types
- `c_src/quiche_erl_nif.c`: C NIF entry points wrapping quiche C API
- Process-per-connection model: each QUIC connection is an Erlang process owning a NIF resource

## Conventions

### Erlang

- 2-space indentation, 100 char line limit
- Function clauses over `case` expressions
- No `if` expressions, no `-import`, no `-compile(export_all)`
- `-spec` on every exported function
- `-opaque` types for NIF resource handles
- Records for internal state only, maps for public API options
- Variable names CamelCase, atoms/functions snake_case
- State records named `#mod_state{}` with `-type state() :: #mod_state{}`
- No process dictionary, no `throw`/`catch` for control flow
- `erlang:send_after/3` for timers, never the `timer` module
- `logger` for logging, never `io:format`

### NIF

- Every C function must check all inputs; a NIF crash kills the entire VM
- `ErlNifResourceType` with destructors for all native state
- `ErlNifMutex` for mutable shared state
- `enif_monitor_process` for owner-based resource cleanup
- `ERL_NIF_OPT_DELAY_HALT` set in load callback (OTP 26+)
- Dirty CPU scheduler for operations >1ms
- Dirty IO scheduler for blocking I/O
- Pre-allocate atoms in load callback, reuse across calls
- Tagged tuple returns: `{ok, Value} | {error, Reason}`
- `enif_make_badarg(env)` for invalid arguments

### Testing

- EUnit for unit tests of pure Erlang functions
- Common Test for integration tests requiring process setup
- PropEr for property-based tests (no-crash on garbage input)
- Test graceful shutdown (`ERL_NIF_OPT_DELAY_HALT` verification)

## Git

- Branch naming: `FGE-{number}` for tickets
- Commit messages must include `Fixes FGE-{number}`
- Run before pushing: `rebar3 compile && rebar3 dialyzer && rebar3 xref && rebar3 ct`
