load("@build_bazel_rules_nodejs//:index.bzl", "pkg_npm", "npm_package_bin", "copy_to_bin")
load("@npm//@bazel/typescript:index.bzl", "ts_project")
load("//tools:jest.bzl", "jest_test")


PACKAGE_NAME = "intl-utils"

pkg_npm(
    name = PACKAGE_NAME,
    srcs = ["package.json"],
    deps = [":lib"],
)

SRCS = glob(["src/*.ts"]) + ['index.ts']

SRC_DEPS = [
]

ts_project(
    name = "lib",
    package_name = "@formatjs/%s" % PACKAGE_NAME,
    srcs = SRCS,
    declaration = True,
    declaration_map = True,
    source_map = True,
    tsconfig = "//:tsconfig.json",
    visibility = ["//visibility:public"],
    deps = SRC_DEPS,
)

jest_test(
    name = "test",
    srcs = [
        "//:tsconfig.json",
    ] + SRCS + glob([
        "tests/**/*.ts",
        "tests/**/*.tsx",
        "tests/**/*.snap",
    ]),
    jest_config = ":jest.config.js",
    tags = [
        # Need to set the pwd to avoid jest needing a runfiles helper
        # Windows users with permissions can use --enable_runfiles
        # to make this test work
        "no-bazelci-windows",
        # TODO: why does this fail almost all the time, but pass on local Mac?
        "no-bazelci-mac",
    ],
    deps = [
        "@npm//@jest/transform",
        "@npm//ts-jest",
        "@npm//@types/jest",
    ] + SRC_DEPS,
)

# API Extractor
copy_to_bin(
    name="setup-api-extractor",
    srcs = ["api-extractor.json", "tsconfig.json", "package.json"]
)

npm_package_bin(
    name = "rollup-dts",
    outs = ["%s.d.ts" % PACKAGE_NAME],
    args = [
        "run",
        "--local",
        "--diagnostics",
        "-c",
        "$(@D)/api-extractor.json"
    ],
    data = [
        ":setup-api-extractor",
        "//:setup-api-extractor",
        ":lib"
    ],
    tool="@npm//@microsoft/api-extractor/bin:api-extractor",
    package = "api-extractor",
)