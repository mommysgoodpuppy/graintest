# Grain Test Framework

A comprehensive testing framework for Grain, inspired by Deno Test.

## Features

- **Simple test registration** with `test()`, `skip()`, `only()`
- **BDD-style API** with `describe()`, `it()`, `xit()`, `fit()`
- **Rich assertions** - `assertEquals`, `assertTrue`, `assertContains`, and more
- **Test hooks** - `beforeAll`, `afterAll`, `beforeEach`, `afterEach`
- **Assertion combinators** - `all()`, `andThen()` for chaining
- **Fail-fast mode** - stop on first failure with `runTestsFailFast()`
- **Multiple reporters** - Pretty, Dot, Compact
- **Test discovery** - find `*_test.gr` files (via wasmtime)
- **Test filtering** - run specific tests by name
- **Result-based error handling** - no exceptions, clean error reporting

## Comparison to Deno Test

| Deno Feature           | Grain Framework        | Status              |
| ---------------------- | ---------------------- | ------------------- |
| `Deno.test()`          | `test()`               | ✅                  |
| `ignore` option        | `skip()`               | ✅                  |
| `only` option          | `only()`               | ✅                  |
| `beforeAll/afterAll`   | `beforeAll/afterAll`   | ✅                  |
| `beforeEach/afterEach` | `beforeEach/afterEach` | ✅                  |
| BDD `describe/it`      | `describe/it/xit/fit`  | ✅                  |
| `--filter`             | `runTestsFiltered()`   | ✅                  |
| `--fail-fast`          | `runTestsFailFast()`   | ✅                  |
| `--reporter`           | `Pretty/Dot/Compact`   | ✅                  |
| Test discovery         | `discoverTestFiles()`  | ✅ (wasmtime)       |
| Test steps             | -                      | ❌                  |
| Coverage               | -                      | ❌ (N/A in Grain)   |
| Mocking/Spying         | -                      | ❌                  |
| Sanitizers             | -                      | ❌ (N/A - no async) |

## Quick Start

### Running Tests

```bash
# Run a specific test file
grain tests/math_test.gr

# Run with filesystem permissions (if needed)
grain --dir . tests/math_test.gr
```

### Writing Your First Test

```grain
module MyTest

from "./lib/test.gr" include Test
use Test.*

test("addition works", () => {
  assertEquals(4, 2 + 2)
})

test("strings work", () => {
  assertContains("Hello World", "World")
})

let results = runTests()
printResults(results)
```

## API Reference

### Test Registration

```grain
from "./lib/test.gr" include Test
use Test.*

// Basic test
test("test name", () => {
  assertEquals(expected, actual)
})

// Skip a test
skip("skipped test", () => {
  // This won't run
})

// Run only this test (others will be skipped)
only("focused test", () => {
  assertTrue(true)
})
```

### BDD-Style API

```grain
from "./lib/test.gr" include Test
from "./lib/bdd.gr" include Bdd
use Test.{ assertEquals, assertTrue }
use Bdd.*

describe("Calculator", () => {
  describe("addition", () => {
    it("adds two numbers", () => {
      assertEquals(4, 2 + 2)
    })
  })

  describe("subtraction", () => {
    it("subtracts two numbers", () => {
      assertEquals(3, 5 - 2)
    })

    xit("skipped test", () => {
      // Won't run
    })
  })
})

let results = runSuite()
printResults(results)
```

### Assertions

All assertions return `Result<Void, String>` - `Ok(void)` on success,
`Err(message)` on failure.

| Assertion                              | Description                             |
| -------------------------------------- | --------------------------------------- |
| `assertEquals(expected, actual)`       | Check equality                          |
| `assertNotEquals(unexpected, actual)`  | Check inequality                        |
| `assertStrictEquals(expected, actual)` | Check physical equality                 |
| `assertTrue(value)`                    | Check value is `true`                   |
| `assertFalse(value)`                   | Check value is `false`                  |
| `assertSome(option)`                   | Check `Option` is `Some(_)`             |
| `assertNone(option)`                   | Check `Option` is `None`                |
| `assertOk(result)`                     | Check `Result` is `Ok(_)`               |
| `assertErr(result)`                    | Check `Result` is `Err(_)`              |
| `assertExists(option)`                 | Alias for `assertSome`                  |
| `assertThat(cond, msg)`                | Custom condition with message           |
| `assertMatches(val, pred, msg)`        | Check value matches predicate           |
| `assertIsSome(option)`                 | Check `Option` is `Some(_)`             |
| `assertIsNone(option)`                 | Check `Option` is `None`                |
| `assertIsOk(result)`                   | Check `Result` is `Ok(_)`               |
| `assertIsErr(result)`                  | Check `Result` is `Err(_)`              |
| `assertContains(haystack, needle)`     | Check string contains substring         |
| `assertStartsWith(str, prefix)`        | Check string starts with prefix         |
| `assertEndsWith(str, suffix)`          | Check string ends with suffix           |
| `assertGreater(left, right)`           | Check `left > right`                    |
| `assertGreaterOrEqual(left, right)`    | Check `left >= right`                   |
| `assertLess(left, right)`              | Check `left < right`                    |
| `assertLessOrEqual(left, right)`       | Check `left <= right`                   |
| `assertListLength(list, expected)`     | Check list length                       |
| `assertArrayLength(arr, expected)`     | Check array length                      |
| `assertSnapshot(value, snapshot)`      | Compare `toString(value)` with snapshot |

### Assertion Combinators

```grain
// Run multiple assertions, fail on first error
test("multiple checks", () => {
  all([
    assertTrue(5 > 3),
    assertEquals(4, 2 + 2),
    assertContains("hello", "ell"),
  ])
})

// Chain assertions sequentially
test("chained assertions", () => {
  andThen(
    assertTrue(true),
    () => andThen(
      assertEquals(4, 2 + 2),
      () => assertGreater(10, 5)
    )
  )
})
```

### Test Hooks

```grain
from "./lib/test.gr" include Test
use Test.*

// Run once before all tests
beforeAll(() => {
  print("Setting up...")
})

// Run once after all tests
afterAll(() => {
  print("Tearing down...")
})

// Run before each test
beforeEach(() => {
  print("Before test")
})

// Run after each test
afterEach(() => {
  print("After test")
})

test("my test", () => {
  assertTrue(true)
})

let results = runTests()
printResults(results)
```

### Filtering Tests

```grain
// Run only tests matching a filter
let results = runTestsFiltered("math")
printResults(results)
```

## Example Test Output

```
--- Test Results ---

[PASS] addition works correctly
[PASS] subtraction works correctly
[FAIL] this test fails
       Expected: 42
       Actual: 2
[SKIP] skipped test (Test marked as ignore)

--------------------
Passed: 2
Failed: 1
Skipped: 1
Total: 4

Some tests failed!
```

## Project Structure

```
graintest/
├── lib/
│   ├── test.gr       # Core test framework
│   ├── bdd.gr        # BDD-style API (describe/it)
│   └── runner.gr     # CLI test discovery
├── tests/
│   ├── math_test.gr      # Math tests example
│   ├── string_test.gr    # String tests example
│   ├── hooks_test.gr     # Hooks demo
│   ├── bdd_test.gr       # BDD API demo
│   └── failure_demo_test.gr  # Failure reporting demo
├── run_tests.gr      # Entry point
└── README.md
```

## Running the Examples

```bash
# Math tests
grain tests/math_test.gr

# String tests
grain tests/string_test.gr

# BDD-style tests
grain tests/bdd_test.gr

# Test hooks demonstration
grain tests/hooks_test.gr

# See failure reporting
grain tests/failure_demo_test.gr

# Run all tests (after compiling runner.wasm)
wasmtime --dir . runner.wasm -- --shell ./tests | sh
```

## Test Discovery & Running All Tests

The CLI runner is **pure Grain** - no external scripts required.

```bash
# 1. Compile the runner once
grain compile lib/runner.gr -o runner.wasm

# 2. Discover tests
wasmtime --dir . runner.wasm -- ./tests

# 3. Run all tests (Unix/macOS)
wasmtime --dir . runner.wasm -- --shell ./tests | sh

# 3. Run all tests (Windows PowerShell)
wasmtime --dir . runner.wasm -- --batch ./tests | iex
```

### CLI Options

```bash
# Show help
wasmtime --dir . runner.wasm -- --help

# Filter test files
wasmtime --dir . runner.wasm -- -f math ./tests

# Output modes
--list     # One file per line (for scripting)
--shell    # Generate bash/sh script
--batch    # Generate PowerShell script
```

### Why wasmtime?

Grain compiles to WebAssembly. The `grain` CLI doesn't support `Fs.readDir` for
directory scanning, but `wasmtime` does when given `--dir .` permission.

## Why Result-Based Assertions?

Grain currently doesn't support exception catching, so the framework uses
`Result<Void, String>` for all assertions. This provides:

1. **Explicit error handling** - no surprises
2. **Composable assertions** - chain with `andThen()` or collect with `all()`
3. **Clear error messages** - failures include context

## License

MIT
