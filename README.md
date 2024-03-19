# Bazel `java_binary` missing runfiles reproduction case

Reproduces a problem with runfiles missing for java_binary data dependencies.

- <https://github.com/mbland/bazel-java-binary-runfiles-repro>

## NOTE: This repo currently doesn't reproduce the intended behavior

It currently tries to reproduce the bug observed with a `go_binary` target, but
using a `sh_binary` target to simplify the problem. However, in this
implementation, the current Bazel release (7.1.0) _also_ doesn't produce a
runfiles directory for the `java_binary` unless that target is built and run
directly. So the behavior described below might not be introducing a bug, but
actually fixing one.

Bazel 7.1.0, 8.0.0-pre.20240128.3, and 8.0.0-pre.20240226.1 all exhibit the
following behavior:

```sh
$ bazel clean --expunge
[...snip...]

$ bazel run //:repro
[...snip...]
INFO: Analyzed target //:repro (79 packages loaded, 1387 targets configured).
[...snip...]
INFO: Found 1 target...       
Target //:repro up-to-date:                                                     
  bazel-bin/repro                                                               
INFO: Elapsed time: 8.155s, Critical Path: 3.04s
INFO: 10 processes: 6 internal, 2 darwin-sandbox, 2 worker.              
INFO: Build completed successfully, 10 total actions
INFO: Running command line: bazel-bin/repro
.../bazel-out/.../bin/NeedsRunfiles: Cannot locate runfiles directory. (Set
$JAVA_RUNFILES to inhibit searching.)

$ bazel run //:NeedsRunfiles
INFO: Analyzed target //:NeedsRunfiles (78 packages loaded, 1381 targets configured).
INFO: Found 1 target...
Target //:NeedsRunfiles up-to-date:
  bazel-bin/NeedsRunfiles
  bazel-bin/NeedsRunfiles.jar
INFO: Elapsed time: 0.339s, Critical Path: 0.02s
INFO: 4 processes: 4 internal.
INFO: Build completed successfully, 4 total actions
INFO: Running command line: bazel-bin/NeedsRunfiles
java_binary runfiles found.

$ bazel run //:repro        
INFO: Analyzed target //:repro (1 packages loaded, 6 targets configured).
INFO: Found 1 target...
Target //:repro up-to-date:
  bazel-bin/repro
INFO: Elapsed time: 0.067s, Critical Path: 0.00s
INFO: 1 process: 1 internal.
INFO: Build completed successfully, 1 total action
INFO: Running command line: bazel-bin/repro
java_binary runfiles found.
```

## What this repo intends to reproduce

As of Bazel 8.0.0-pre.20240226.1, depending on a `java_binary` target as a data
dependency stopped producing a runfiles directory, resulting in the following
error:

```sh
[... snip ...]/NeedsRunfiles: Cannot locate runfiles directory. (Set $JAVA_RUNFILES to inhibit searching.)
```

In the bazelbuild/bazel repo, gitk 8.0.0-pre.20240128.3..8.0.0-pre.20240226.1
shows several commits touching runfiles, but these two seem the most suspicious
to me:

- cff79d0aa6f2d470b8648f2b89beeefd0867e177: Do not build the runfiles tree when
  creating deploy jars.
-RRRRRRRRbf2dc2214ffe6abc807e4f597175e1a6c: Do not add the runfiles middleman to
  the inputs of the deploy jar action.
