# Repository Guidelines

## Project Structure & Module Organization
Markflow is a Cargo workspace with `crates/markflow-core`, `bindings/markflow-node`, `bindings/markflow-wasm`, and `tools/markflow-cli`. `markflow-core` owns the zero-copy pipeline (Input → Parser → Adapter → lol_html → Output) and stays FFI-free. `markflow-node` binds the core via napi-rs ThreadsafeFunctions so Node 18+ clients can `for await` streamed chunks. `markflow-wasm` uses wasm-bindgen and should export only the `ReadableStream` surface to keep bundles tight. `markflow-cli` replays stdin through the core for dogfooding and perf baselines.

## Build, Test, and Development Commands
- `cargo fmt && cargo clippy --workspace --all-targets`: enforce formatting plus zero-copy lint gates.
- `cargo test -p markflow-core`: run core unit tests; fixtures live in `crates/markflow-core/tests/`.
- `cargo run -p markflow-cli -- --file samples/large.md`: replay Markdown through the streaming pipeline.
- `pnpm -C bindings/markflow-node test`: compile napi artifacts and run TypeScript smoke tests.
- `wasm-pack test --headless --chrome bindings/markflow-wasm`: verify the browser `ReadableStream` contract end to end.

## Coding Style & Naming Conventions
Rust uses 4-space indentation, modules in `snake_case`, public types/traits in `CamelCase`, and feature flags prefixed `streaming-*`. Prefer `&[u8]` or `Bytes` over `Vec<u8>` to honor the zero-copy goal, and document every unavoidable allocation inline. Node bindings stay in TypeScript with PascalCase classes (`MarkflowStream`) and camelCase methods; run `rustfmt` before committing regenerated bindings.

## Testing Guidelines
Add chunked fixtures that feed the parser byte-by-byte so `StreamingCursor` never clones buffers. Node tests should assert that offsets increase monotonically across `for await` iterations and backpressure resolves promptly. Wasm tests can wrap the exported stream with `new Response(stream).text()` to confirm ordering in browsers. Use `cargo tarpaulin -p markflow-core` for coverage and keep adapter-heavy modules above 85%.

## Commit & Pull Request Guidelines
Follow Conventional Commits with scoped prefixes such as `feat(core):`, `fix(node):`, `chore(wasm):`, and `test(cli):`, keeping the first line ≤ 72 characters. Reference issues (`Closes #42`) and describe the dataset used to validate zero-copy streaming. Every PR should summarize touched crates, include benchmark snippets (`cargo bench streaming_parser`), and paste results from Node/Wasm smoke tests. Changes that cross the FFI boundary need reviewers from both the core and binding teams.

## Security & Performance Notes
Never log customer Markdown; redact to byte counts or hashes instead. Validate buffers before injecting them into lol_html adapters to avoid UB from invalid UTF-8. When a TODO in Markdown or Rust notes a performance follow-up, link to the tracking issue so agents can prioritize the zero-copy roadmap.
