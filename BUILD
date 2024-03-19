load("@rules_java//java:defs.bzl", "java_binary")

package(default_visibility = ["//visibility:public"])

java_binary(
    name = "NeedsRunfiles",
    main_class = "com.mike_bland.NeedsRunfiles",
    runtime_deps = [
        "//src/com/mike_bland:needsrunfiles",
    ],
)

sh_library(
    name = "reprolib",
    srcs = [],
    data = [
        ":NeedsRunfiles",
        ":NeedsRunfiles.jar",
    ]
)

sh_binary(
    name = "repro",
    srcs = ["repro.sh"],
    deps = [
        ":reprolib",
        "@bazel_tools//tools/bash/runfiles"
    ],
)
