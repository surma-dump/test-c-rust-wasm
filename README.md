# Rust+C in the same WASM binary

I created this repository to test the current state of the Rust nightly compiler for producing a compatible C ABI for the `wasm32-unknown-unknown` target with `--Z wasm_c_abi=spec`.

For context, this is the [relevant tracking issue](https://github.com/rustwasm/wasm-bindgen/issues/3454) in `wasm-bindgen`.

This project contains a minimal example of producing a single WASM binary with both C and Rust code that can call each other.

It's a simple calculator where:

- `add()` is defined by C
- `subtract()` is defined by Rust
- `multiply()` is defined by Rust but calls `add()` in C multiple times
- `divide()` is defined by C but calls `subtract()` in Rust multiple times

All methods are exported from a single `.wasm` file and can be called from JS.

## Building

To build the project, just call `./build_both.sh`. You need llvm, clang, and Rust nightly.

To see it working in your browser, use your preferred local server:

```bash
npm install -g serve
serve
```

Then, visit http://localhost:3000/wasm and check the console.

## How it works

You can read the mini build scripts `build_c.sh`, `build_rust.sh`, and `build_both.sh` to check the commands needed to do it. You'll need LLVM and Rust nightly.

The main idea behind the scripts is simple:

- Transpile C to LLVM internal representation.
- Transpile Rust to LLVM internal representation. Requires Rust nightly with `--Z wasm_c_abi=spec`.
- Compile all `.ll` files with LLVM compiler `llc`.
- Link the compiled files with `wasm-ld`.

## Next steps

I plan to eventually introduce more complex examples with multiple files and import external C and Rust libraries.

## References

I went down a deep rabbit hole and read many articles, forums, GH issues, PRs, and threads before arriving at a solution, and I won't mention them all here.

One article deserves special mention, though.

The main idea to transpile the code to LLVR IR and the steps to do it for C came from [Compiling C to WebAssembly without Emscripten](https://dassur.ma/things/c-to-webassembly) by [@surma](https://github.com/surma).

The funny thing is he mentioned this intermediate LLVM IR step just for the sake of completeness. He could very well have left it unmentioned, and he even writes that nobody uses it in practice.

Well, I used it for this very specific case, so thank you [@surma](https://github.com/surma) for mentioning it anyway!

