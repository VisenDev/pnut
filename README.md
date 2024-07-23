# Pnut: A Self-Compiling C Transpiler Targeting Human-Readable POSIX Shell

Pnut compiles a reasonnably large subset of C99 to human-readable POSIX shell
scripts. It can be used to write portable shell scripts without having to write
shell.

It's main uses are:

- As a transpiler to write portable shell scripts in C.
- As a way to bootstrap a compiler written in C with an executable version that is still human readable (See [reproducible builds](#reproducible-builds)).

Main features:

- No new language to learn -- C code in, shell code out.
- The human-readable shell script is easy to read and understand.
- A runtime library including file I/O and dynamic memory allocations.
- A preprocessor (`#include`, `#ifdef`, `#define MACRO ...`, `#define MACRO_F(x) ...`).
- Integrates easily with existing shell scripts.

The [examples](examples/compiled) directory contains a many example.
We invite you take a look!

## Install

Pnut can be distributed as the `pnut.sh` shell script, or compiled to executable
code using a C compiler.

To install pnut:

```shell
> git clone https://github.com/udem-dlteam/pnut.git
> cd pnut
> sudo make install
```

This installs both `pnut.sh` and `pnut` in `/usr/local/bin`.

### Compilation options

Certain compilation options can be used to change the generated shell script:

- `-DRT_COMPACT` reduces the size of the runtime library at the
  cost of reduced I/O performance.
- `-DSH_SAVE_VARS_WITH_SET` reduces the overhead of local variables at the cost
  of readability. This can reduce the execution time of certain programs by more
  than 50%.
- `-DSH_INCLUDE_C_CODE` includes the original C code in the generated shell
  script.

These can be set using `make install BUILD_OPT="..."`.

## How to use

The `pnut` compiler takes a C file path as input, and outputs to stdout the
POSIX shell code.

Here's an example of how to compile a C file using Pnut:

```shell
> pnut.sh examples/fib.c > fib.sh # Compile fib.c to a shell script
> chmod +x fib.sh                 # Make the shell script executable
> ./fib.sh                        # Run the shell script
```

## Which shell to use

Because Pnut generates purely POSIX shell code, the generated shell scripts
can be run on any POSIX compliant shell. However, certain shells are faster than
others. For faster scripts, we recommend the use of `ksh`, `dash` or `bash`.
`zsh` is also supported but tends to be slower on large programs.

## Reproducible builds

Because Pnut can be distributed as a human-readable shell script (`pnut.sh`), it
can serve as the basis for a reproducible build system. With a POSIX compliant
shell, `pnut.sh` is sufficiently powerful to compile itself and, with some
effort, [TCC](https://bellard.org/tcc/). Because TCC can be used to bootstrap
GCC, this makes it possible to bootstrap a fully featured build toolchain from
only human-readable source files and a POSIX shell.

Because Pnut doesn't support certain C features used in TCC, Pnut features a
native code backend that supports a larger subset of C99. We call this compiler
`pnut-exe`, and it can be compiled using `pnut.sh`. This makes it possible to
compile `pnut-exe.c` using `pnut.sh`, and then compile TCC, all from a POSIX
shell.

## Limitations

Unfortunately, certains C constructs don't map nicely to POSIX shell which means:

- No support for floating point numbers and unsigned integers.
- `goto` and `switch` fallthrough are not supported.
- The address of (`&`) operator on local variables is not supported.

## Known issues

- The preprocessor is not perfect and may fail on some edge cases. `#if` and `#elif` are not supported. `#include <...>` are ignored.
- All local variable declarations must be at the beginning of a function.
- Aggregate types (arrays and structures) cannot be stack-allocated, passed by value or nested in a structure.
- `do { ... } while(...)` is not supported at the moment.

## Contributing

Pnut is a research project and contributions are welcome. Please open an issue
to report any bugs or to discuss new features.

To make sure your changes are good, a good practice is to attempt the bootstrap
of `pnut.c` using `pnut.sh`. This can be done using the `./bootstrap-pnut.sh`
command. Using `ksh`, this should take around 30s.
