load("@rules_pkg//:pkg.bzl", "pkg_tar")
load("//scripts/docs:jekyll.bzl", "jekyll_build")
load("//scripts/docs:doc_versions.bzl", "DOC_VERSIONS")

exports_files(
    [
        "docs/user-manual.html",
        "command-line-reference-prefix.html",
        "command-line-reference-suffix.html",
    ],
)

exports_files(
    [
        "images/favicon.ico",
    ],
    visibility = ["//src/main/native/windows:__pkg__"],
)

DOT_GRAPH_HTML_FILES = [
    "docs/query.html",
    "docs/build-ref.html",
]

filegroup(
    name = "srcs",
    srcs = glob(["**"]),
    visibility = ["//:__pkg__"],
)

# Required to move the shared CSS Files from the external repo's execroot location
# to the _sass location expected by the local css/main.scss imports.
genrule(
    name = "style-common",
    srcs = ["@bazel_website//:_sass/style.scss"],
    outs = ["_sass/style.scss"],
    cmd = "cp $< $@",
)

filegroup(
    name = "jekyll-srcs",
    srcs = glob(
        ["**/*"],
        exclude = [
            "BUILD",
            "jekyll-tree.sh",
            "*.swp",
            "command-line-reference-prefix.html",
            "command-line-reference-suffix.html",
            "site/README.md",
        ] + DOT_GRAPH_HTML_FILES,
    ) + [
        ":style-common",
    ],
)

pkg_tar(
    name = "jekyll-files",
    srcs = [":jekyll-srcs"],
    strip_prefix = ".",
)

pkg_tar(
    name = "bootstrap-css",
    srcs = ["//third_party/css/bootstrap:bootstrap_css"],
    package_dir = "assets",
    strip_prefix = "/third_party/css/bootstrap",
)

pkg_tar(
    name = "bootstrap-images",
    srcs = ["//third_party/css/bootstrap:bootstrap_images"],
    package_dir = "assets",
    strip_prefix = "/third_party/css/bootstrap",
)

pkg_tar(
    name = "font-awesome-css",
    srcs = ["//third_party/css/font_awesome:font_awesome_css"],
    package_dir = "assets",
    strip_prefix = "/third_party/css/font_awesome",
)

pkg_tar(
    name = "font-awesome-font",
    srcs = ["//third_party/css/font_awesome:font_awesome_font"],
    package_dir = "assets",
    strip_prefix = "/third_party/css/font_awesome",
)

pkg_tar(
    name = "bootstrap-js",
    srcs = ["//third_party/javascript/bootstrap:bootstrap_js"],
    package_dir = "assets",
    strip_prefix = "/third_party/javascript/bootstrap",
)

pkg_tar(
    name = "jekyll-base",
    deps = [
        ":bootstrap-css",
        ":bootstrap-images",
        ":bootstrap-js",
        ":dot-graphs",
        ":font-awesome-css",
        ":font-awesome-font",
        ":jekyll-files",
    ],
)

pkg_tar(
    name = "skylark-rule-docs",
    srcs = [
        "//tools/build_defs/pkg:README.md",
    ],
    strip_prefix = "/tools/build_defs",
)

genrule(
    name = "dot-graphs",
    srcs = DOT_GRAPH_HTML_FILES,
    outs = ["dot.tar"],
    cmd = """
origdir=$$PWD
tmpdir=$$(mktemp -d)
for f in $(SRCS); do
    mkdir -p $$tmpdir/$$(dirname $$f)
    if which dot > /dev/null; then
      $(location //scripts/docs:generate_dot_graphs) < $$f > $$tmpdir/$$f
    else
      cp $$f $$tmpdir/$$f
    fi
done
cd $$tmpdir/site
tar cf $$origdir/$@ *
""",
    tools = ["//scripts/docs:generate_dot_graphs"],
)

# Generate the documentation tree in the current git worktree
genrule(
    name = "jekyll-tree",
    srcs = [
        ":jekyll-base",
        ":skylark-rule-docs",
        "//src/main/java/com/google/devtools/build/lib:gen_buildencyclopedia",
        "//src/main/java/com/google/devtools/build/lib:gen_skylarklibrary",
        "//src/main/java/com/google/devtools/build/lib:gen_command-line-reference",
        "//tools/build_defs/repo:doc",
    ],
    outs = ["jekyll-tree.tar"],
    cmd = ("$(location jekyll-tree.sh) $@ " +
           "$(location :jekyll-base) " +
           "$(location :skylark-rule-docs) " +
           "$(location //src/main/java/com/google/devtools/build/lib:gen_buildencyclopedia) " +
           "$(location //src/main/java/com/google/devtools/build/lib:gen_skylarklibrary) " +
           "$(location //src/main/java/com/google/devtools/build/lib:gen_command-line-reference) " +
           "$(location //tools/build_defs/repo:doc)"),
    tools = [
        "jekyll-tree.sh",
    ],
)

jekyll_build(
    name = "site",
    srcs = [
        "@jekyll_tree_%s//file" % DOC_VERSION["version"].split("-")[0].replace(".", "_")
        for DOC_VERSION in DOC_VERSIONS
    ] + [":jekyll-tree"],  # jekyll-tree *must* come last to use layouts and includes from master
    bucket = "docs.bazel.build",
)

# A smaller site build target. Contains just the docs for latest released version and master.
jekyll_build(
    name = "site_latest",
    srcs = [
        "@jekyll_tree_%s//file" % DOC_VERSIONS[0]["version"].split("-")[0].replace(".", "_"),
        ":jekyll-tree",  # jekyll-tree *must* come last to use layouts and includes from master
    ],
)
