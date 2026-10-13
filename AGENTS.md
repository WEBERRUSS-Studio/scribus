# AGENTS.md

Working rules for agents (and humans) developing in this fork. Upstream Scribus
rules still apply to anything intended to go back upstream.

## What this checkout is

Scribus — C++ desktop publishing, Qt 6 + CMake, ~22 300 commits, ~7 900 tracked files.

* Upstream is `svn://scribus.net/trunk/Scribus` (stated in `README.md`).
  Bugs and patches go to the Scribus Bug Tracker <http://bugs.scribus.net>; the
  official GitHub repository is a mirror only, and PRs there get forwarded to the
  tracker.
* `origin` = `https://github.com/WEBERRUSS-Studio/scribus` — this fork, used for
  further development and experimentation.
* Measured 2026-10-13: `master` is a **pure upstream snapshot** — in the last 500
  commits author == committer == upstream developer (Craig Bradney, Jean Ghali), and
  `master` == `origin/master`. The fork carries no own commits yet.

Consequence: `master` is the upstream tracking branch. Fork work lives on its own
branch; `master` moves only by an upstream sync or a deliberate merge decision.

## Where work happens

| Path | Note |
| --- | --- |
| `scribus/` | the application — most work happens here |
| `scribus/plugins/import/`, `scribus/plugins/export/` | file-format filters. Each is optional at configure time: verify a filter with its library installed, or report it as unverified |
| `scribus/tests/` | C++ test executables (see Verification) |
| `CMakeLists.txt`, `ConfigureChecks.cmake`, `cmake/modules/` | build system; a change here must keep a plain out-of-source configure working |
| `resources/` | UI, icons, translations (`.po`); sample documents under `resources/tests/` |

## Task source

1. **Upstream bug** — the work item is a bug id on <http://bugs.scribus.net>. Commit
   style in that case is `#<bug id>: <summary>`, as observed in history:
   `969c3fe4b #17995: Baseline alignement does not work correctly in tables`.
2. **Fork experiment or own feature** — the fork has no hosted board. The branch name
   and the commit message carry the intent; there is no local task file to update, and
   adding one is a project decision, not an agent default.

Ask for the bug id or the experiment goal before implementing. Never invent an id.

## Branches, commits, integration

* One branch per change, off `master`, prefixed `feat/ fix/ docs/ chore/ sync/`
  (the local wt-pilot tooling derives branch names from these types).
* Commit message: `#<bug id>: <summary>` for tracker fixes, otherwise imperative mood.
  No `#`-prefixed id unless the id is real.
* Pushing the branch to `origin` (the fork) is the normal integration point. Merging
  into `master` is a separate decision, never an automatic consequence of a finished
  branch, and never done on a branch that has not passed Verification below.
* Sync from upstream: merge into a `sync/` branch rather than rebasing a branch others
  may already have built on. Adding upstream as a remote changes this checkout's
  configuration — ask first.

## Verification

Build out of source, so the checkout stays clean and several configurations coexist:

```sh
cmake -S . -B ../scribus-build -G Ninja -DCMAKE_BUILD_TYPE=Release
cmake --build ../scribus-build -j"$(nproc)"
```

Measured 2026-10-13 on this machine (Qt 6.11.2, CMake + Ninja, ccache, 24 cores):
configure succeeds, 1157 build targets, full build in ~6 min, binary at
`<build-dir>/scribus/scribus`. `librevenge-0.0`, `libwpg-0.2` and `libmspub-0.0` are
not installed, so the FreeHand / WordPerfect / MS Publisher import filters are
skipped. That is not a failure, but a filter change must be built where its library
exists.

Configure also writes `scribus.desktop` and `scribus.appdata.xml` into the **source**
directory even for an out-of-source build (`CMakeLists.txt:597` and `CMakeLists.txt:616`
use `${CMAKE_CURRENT_SOURCE_DIR}` as the output). They are untracked build artifacts —
never commit them.

Automated tests do **not** work here — do not report a test run:

* `WITH_TESTS` is referenced (`scribus/CMakeLists.txt:45`) but never declared as a
  CMake option, so it is off unless passed explicitly.
* With `-DWITH_TESTS=1` the test sources compile, but linking fails under Qt 6
  (measured 2026-10-13): `scribus/tests/CMakeLists.txt:32` builds the test link line
  from `${QT_QTTEST_LIBRARY} ${QT_LIBRARIES}`, and neither variable is defined
  anywhere in the Qt 6 configuration — the executable is linked against no Qt library
  (`undefined reference to QTest::qVerify`).
* `scribus/tests/CMakeLists.txt:38` registers `add_test(...)`, but the root
  `CMakeLists.txt:108` has `enable_testing()` commented out, so `ctest` finds no test
  even if the link were fixed.

Until that link line is repaired, the check for a change is: configure succeeds, the
touched target compiles, and the built app is run for anything with visible
behaviour. State what was not run.
A change with visible behaviour additionally requires running the built app and
exercising the affected action — a clean compile is not proof of behaviour. Report
what was not run.

## UI / UX

Design authority stays with the maintainer. Implement the agreed design faithfully;
a new or changed layout, metric, colour or interaction needs explicit approval — an
agent's own reasoning never authorizes a design change. A missing state or breakpoint
in the design is a question to ask, not something to improvise.

## Language

Repository language is English — documentation, commit messages, code comments.
Conversation with the maintainer may be German.
