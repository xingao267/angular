workspace(name = "angular")

load("@bazel_tools//tools/build_defs/repo:http.bzl", "http_archive")

# Uncomment for local bazel rules development
#local_repository(
#    name = "build_bazel_rules_nodejs",
#    path = "../rules_nodejs",
#)
#local_repository(
#    name = "npm_bazel_typescript",
#    path = "../rules_typescript",
#)

# Fetch rules_nodejs so we can install our npm dependencies
http_archive(
    name = "build_bazel_rules_nodejs",
    sha256 = "1fbea5f33a8cbf8fd798f4cf7973a9b09e53bda872c75b5665a92a6d28fbbb13",
    urls = ["https://github.com/bazelbuild/rules_nodejs/releases/download/0.27.10/rules_nodejs-0.27.10.tar.gz"],
)

# Check the bazel version and download npm dependencies
load("@build_bazel_rules_nodejs//:defs.bzl", "check_bazel_version", "check_rules_nodejs_version", "node_repositories", "yarn_install")

# Bazel version must be at least v0.21.0 because:
#   - 0.21.0 Using --incompatible_strict_action_env flag fixes cache when running `yarn bazel`
#            (see https://github.com/angular/angular/issues/27514#issuecomment-451438271)
check_bazel_version(
    message = """
You no longer need to install Bazel on your machine.
Angular has a dependency on the @bazel/bazel package which supplies it.
Try running `yarn bazel` instead.
    (If you did run that, check that you've got a fresh `yarn install`)

""",
    minimum_bazel_version = "0.21.0",
)

# The NodeJS rules version must be at least v0.15.3 because:
#   - 0.15.2 Re-introduced the prod_only attribute on yarn_install
#   - 0.15.3 Includes a fix for the `jasmine_node_test` rule ignoring target tags
#   - 0.16.8 Supports npm installed bazel workspaces
#   - 0.26.0 Fix for data files in yarn_install and npm_install
check_rules_nodejs_version("0.26.0")

# Setup the Node.js toolchain
node_repositories(
    node_version = "10.9.0",
    package_json = ["//:package.json"],
    preserve_symlinks = True,
    # yarn 1.13.0 under Bazel has a regression on Windows that causes build errors on rebuilds:
    # ```
    # ERROR: Source forest creation failed: C:/.../fyuc5c3n/execroot/angular/external (Directory not empty)
    # ```
    # See https://github.com/angular/angular/pull/29431 for more information.
    # It possible that versions of yarn past 1.13.0 do not have this issue, however, before
    # advancing this version we need to test manually on Windows that the above error does not
    # happen as the issue is not caught by CI.
    yarn_version = "1.12.1",
)

yarn_install(
    name = "npm",
    data = [
        "//:tools/npm/@angular_bazel/index.js",
        "//:tools/npm/@angular_bazel/package.json",
        "//:tools/postinstall-patches.js",
        "//:tools/yarn/check-yarn.js",
    ],
    package_json = "//:package.json",
    # Don't install devDependencies, they are large and not used under Bazel
    prod_only = True,
    yarn_lock = "//:yarn.lock",
)

yarn_install(
    name = "ts-api-guardian_deps",
    package_json = "@angular//tools/ts-api-guardian:package.json",
    yarn_lock = "@angular//tools/ts-api-guardian:yarn.lock",
)

# Install all bazel dependencies of the @npm npm packages
load("@npm//:install_bazel_dependencies.bzl", "install_bazel_dependencies")

install_bazel_dependencies()

# Load angular dependencies
load("//packages/bazel:package.bzl", "rules_angular_dev_dependencies")

rules_angular_dev_dependencies()

# Load karma dependencies
load("@npm_bazel_karma//:package.bzl", "rules_karma_dependencies")

rules_karma_dependencies()

# Setup the rules_webtesting toolchain
load("@io_bazel_rules_webtesting//web:repositories.bzl", "web_test_repositories")

web_test_repositories()

# Temporary work-around for https://github.com/angular/angular/issues/28681
# TODO(gregmagolan): go back to @io_bazel_rules_webtesting browser_repositories
load("@npm_bazel_karma//:browser_repositories.bzl", "browser_repositories")

browser_repositories()

# Setup the rules_typescript tooolchain
load("@npm_bazel_typescript//:index.bzl", "ts_setup_workspace")

ts_setup_workspace()

# Setup the rules_sass toolchain
load("@io_bazel_rules_sass//sass:sass_repositories.bzl", "sass_repositories")

sass_repositories()

# Setup the skydoc toolchain
load("@io_bazel_skydoc//skylark:skylark.bzl", "skydoc_repositories")

skydoc_repositories()

load("@bazel_toolchains//rules:environments.bzl", "clang_env")
load("@bazel_toolchains//rules:rbe_repo.bzl", "rbe_autoconfig")

rbe_autoconfig(
    name = "rbe_ubuntu1604_angular",
    # The sha256 of marketplace.gcr.io/google/rbe-ubuntu16-04 container that is
    # used by rbe_autoconfig() to pair toolchain configs in the @bazel_toolchains repo.
    base_container_digest = "sha256:69c9f1652941d64a46f6f7358a44c1718f25caa5cb1ced4a58ccc5281cd183b5",
    # Needed because CircleCI does not allow mounting from job space to remote containers.
    # https://circleci.com/docs/2.0/building-docker-images/#mounting-folders
    copy_resources = True,
    # Note that if you change the `digest`, you might also need to update the
    # `base_container_digest` to make sure marketplace.gcr.io/google/rbe-ubuntu16-04-webtest:<digest>
    # and marketplace.gcr.io/google/rbe-ubuntu16-04:<base_container_digest> have the
    # same Clang and JDK installed.
    digest = "sha256:ce383089ba3fc1f6f687fc1e492401e21594c91d9587a315e26e7d6c91aafc4f",
    env = clang_env(),
    registry = "marketplace.gcr.io",
    repository = "google/rbe-ubuntu16-04-webtest",
)
