# Update Homebrew Formula

A GitHub Action that synchronizes a Homebrew formula with a GitHub release.

> 🚧 This is still in development, and not ready for general use.

Add this action to a workflow
to update a corresponding [Homebrew formula](https://brew.sh) in your tap repository
whenever you create a new [GitHub release](https://docs.github.com/en/free-pro-team@latest/github/administering-a-repository/about-releases).
Your Homebrew formula will be updated with the tag and revision of the release.
Any assets associated with the release whose name matches the
[expected pattern](https://github.com/NSHipster/update-homebrew-formula-action/blob/e1551f21a97c71feba4202ab613f460f008807cb/entrypoint.rb#L65)
will be added as pre-built binary artifacts,
or [bottles](https://www.rubydoc.info/github/Homebrew/brew/Formula#bottle-class_method).

To better understand what this action does and why it's useful,
consider the following scenario:

* * *

Mona has a project `mona/hello` with source code
for building an executable named `hello`.
To make it easier to install,
Mona provides the following Homebrew [formula](https://docs.brew.sh/Formula-Cookbook)
in a [tap](https://docs.brew.sh/Formula-Cookbook#homebrew-terminology)
that she hosts in a repository named `mona/homebrew-formulae`:

```ruby
class Hello < Formula
  desc "Says hello"
  homepage "https://github.com/mona/hello"
  url "https://github.com/mona/hello.git", tag: "1.0.0", revision: "d95b2990f6186523cda25cea4f9d45bc1fde069f"

  bottle do
    root_url "https://github.com/mona/hello/releases/download/1.0.0"
    sha256 "12a4ffc9c6a598b554b640d1473a2e413a7802ad5d2fbe79d1b2fbe341d0fb0d" => :catalina
  end

  depends_on xcode: ["12.0", :build]

  def install
    system "make", "install", "prefix=#{prefix}"
  end

  test do
    system "#{bin}/hello"
  end
end
```

This allows anyone with Homebrew installed
to install the `hello` command with a single command:

```terminal
$ brew install mona/formulae/hello
```

However, this convenience for the user comes at a cost to Mona
(beyond the fixed cost of creating a formula in the first place).
Whenever Mona wants to release a new version of `hello`,
she must do the following:

- [x] Create and push a new tag
- [x] Create a new release on GitHub
- [x] Build and upload a pre-built binary for the release
- [x] Calculate the SHA256 checksum for the binary
- [x] Update the Homebrew formula with the new tag, revision,
      and asset checksums

If she forgets to do all of these steps
(or makes a mistake),
her users won't get the latest version when they install `hello`.

* * *

This action automates the manual, error-prone process described above,
streamlining the release of any tool you distribute via
your own Homebrew tap.

## Usage

### Inputs

- `repository`:
  **Required.**
  The project repository (e.g. mona/hello).
- `tap`:
  **Required.**
  The Homebrew tap repository (e.g. mona/homebrew-formulae).
- `formula`:
  **Required.**
  The path to the formula in the tap repository (e.g. Formula/hello.rb).

> **Important**:
> This action also requires the `GITHUB_TOKEN` environment variable to be set.

### Example Workflow

```yml
# .github/workflows/release.yml
name: Release

on:
  release:
    types:
      - created

jobs:
  homebrew:
    name: Update Homebrew formula
    runs-on: ubuntu-latest
    steps:
      - uses: NSHipster/update-homebrew-formula-action@main
        with:
          repository: mona/hello
          tap: mona/homebrew-formulae
          formula: Formula/hello.rb
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

We recommend running this action as part of a workflow that triggers on
[release events](https://docs.github.com/en/free-pro-team@latest/actions/reference/events-that-trigger-workflows#release)
with the `created` activity type.
This way, any release that's created —
whether manually or programmatically
(such as with [actions/create-release](https://github.com/actions/create-release)) —
will benefit from the same automation.

## License

MIT
