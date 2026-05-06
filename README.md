# Nova Lang

A hand-crafted compiler for the **Nova** programming language — a statically typed, systems-level DSL that compiles down to native binaries via LLVM IR. Written from scratch in C11.

---

## What is Nova?

Nova is a minimal language designed for clarity and control. It gives you low-level power (manual types, pointers, extern C calls) without the ceremony. The compiler takes `.nova` source files and produces native executables through a clean multi-stage pipeline.

```nova
extern fn printf(fmt: str, val: i64) -> i32;

fn factorial(n: i64) -> i64 {
    if n <= 1 {
        return 1;
    }
    return n * factorial(n - 1);
}

fn main() -> i32 {
    let result: i64 = factorial(10);
    printf("10! = %lld\n", result);
    return 0;
}
```

---

## How It Works

Source code flows through five sequential stages before becoming an executable:

```
.nova source
    │
    ▼
[Lexer]      → token stream (handles literals, keywords, operators)
    │
    ▼
[Parser]     → typed AST (recursive-descent + Pratt expression parsing)
    │
    ▼
[Sema]       → type-checked AST (two-pass: symbol registration → body checking)
    │
    ▼
[CodeGen]    → LLVM IR text (.ll file)
    │
    ▼
 llc / clang → native binary
```

Each stage is implemented in a single focused file under `src/`. There's no magic, no generated code — just C.

---

## The Language

### Built-in Types

| Nova type | Size    | Notes                    |
|-----------|---------|--------------------------|
| `i8–i64`  | 8–64bit | Signed integers          |
| `u8–u64`  | 8–64bit | Unsigned integers        |
| `f32`     | 32bit   | Single-precision float   |
| `f64`     | 64bit   | Double-precision float   |
| `bool`    | 1bit    | `true` / `false`         |
| `str`     | ptr     | String literal (`i8*`)   |
| `*T`      | ptr     | Pointer to any type      |
| `[N]T`    | inline  | Fixed-size array         |

### Control Flow

```nova
// if / else if / else
if score >= 90 {
    puts("A");
} else if score >= 70 {
    puts("B");
} else {
    puts("F");
}

// while loop
while running {
    tick();
}

// range-based for
for i in 0..10 {
    process(i);
}
```

### Variables & Mutability

```nova
let x: i64 = 100;        // immutable
let mut count: i64 = 0;  // mutable
```

### Structs, Enums & Casting

```nova
struct Vec2 { x: f64, y: f64 }

enum Direction { North, South, East, West }

let angle: f64 = 1;
let deg: i64 = angle as i64;
```

### Calling into C

```nova
extern fn malloc(size: u64) -> *u8;
extern fn free(ptr: *u8) -> void;
```

---

## Getting Started

### Compile a Nova file

```powershell
# Step 1 — emit LLVM IR
.\nova_compiler.exe --emit-ir hello.nova -o hello.ll

# Step 2 — compile to native binary
clang hello.ll -o hello.exe

# Step 3 — run it
.\hello.exe
```

Some programs (like `sort.nova`) use a bundled C helper:

```powershell
clang sort.ll helpers.c -o sort.exe
```

### Debug & Inspect

```powershell
# Dump the AST
.\nova_compiler.exe --ast hello.nova

# Dump the token stream
.\nova_compiler.exe --tokens hello.nova
```

### VS Code

Open the repo in VS Code, then press `Ctrl+Shift+B` and pick **"Compile and Run (active file)"**. The task handles IR generation, linking, and execution automatically.

---

## Project Layout

```
nova-lang/
├── src/
│   ├── lexer.c       # Tokenisation
│   ├── parser.c      # AST construction
│   ├── ast.c         # Node types + arena allocator
│   ├── sema.c        # Type checking & symbol resolution
│   ├── codegen.c     # LLVM IR emission
│   ├── diag.c        # Error reporting
│   └── main.c        # Entry point / driver
├── include/          # Headers for each module
├── tests/            # Nova test programs
├── examples/         # Showcase programs
├── docs/             # Extended documentation
├── scripts/          # Build helpers
└── Makefile
```

---

## Memory Model

The compiler allocates everything through a single **arena allocator** (`ast.c`). There is no `malloc`/`free` scattered through the codebase — the entire compilation run uses one contiguous block, which is released at exit. This eliminates heap fragmentation and makes the compiler fast even on large inputs.

---

## Error Reporting

Diagnostics are location-aware and colour-coded in terminals that support it. Colour is suppressed automatically when output is piped or redirected.

---

## Operators

| Category   | Operators              |
|------------|------------------------|
| Arithmetic | `+ - * / %`            |
| Bitwise    | `& \| ^ ~ << >>`       |
| Logical    | `&& \|\| !`            |
| Comparison | `== != < > <= >=`      |
| Assignment | `= += -= *= /=`        |
| Cast       | `as`                   |
| Range      | `..`                   |
