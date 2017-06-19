# Development Guide

First: [How to Setup the Development Environment](development-setup.md)

This document is the canonical source of truth for things like supported
toolchain versions for building Kubernetes.

Please submit an [issue] on Github if you
* Notice a requirement that this doc does not capture.
* Find a different doc that specifies requirements (the doc should instead link
  here).

Development branch requirements will change over time, but release branch
requirements are frozen.

## Pre submit flight checks

Determine whether your issue or pull request is improving Kubernetes'
architecture or whether it's simply fixing a bug.

If you need a diagram, add it.  SEPARATE the description of the problem (e.g. Y
is a critical component that is too slow for an SLA that we care about) from the
solution (e.g. make X faster).

Some of these checks were less common in Kubernetes' earlier days. Now that we
have over 1000 contributors, each issue should be filed with care. No issue
should take more than 5 minutes to check for sanity (even the busiest of
reviewers can spare 5 minutes to review a patch that is thoughtfully justified).

### Is this just a simple bug fix?

Simple bug patches are easy to review since test coverage is submitted with the
patch.  Bug fixes don't usually require a lot of extra testing, but please
update the unit tests so they catch the bug!

### Is this an architecture improvement?

Some examples of "Architecture" improvements include:

- Adding a new feature or making a feature more configurable or modular.
- Improving test coverage.
- Decoupling logic or creation of new utilities.
- Making code more resilient (sleeps, backoffs, reducing flakiness, etc.).

These sorts of improvements are easily evaluated, especially when they decrease
lines of code without breaking functionality.  That said, please explain exactly
what you are 'cleaning up' in your Pull Request so as not to waste a reviewer's
time.

If you're making code more resilient, include tests that demonstrate the new
resilient behavior.  For example: if your patch causes a controller to better
handle inconsistent data, make a mock object which returns incorrect data a few
times and verify the controller's new behaviour.

### Is this a performance improvement ?

Performance bug reports MUST include data that demonstrates the bug.  Without
data, the issue will be closed.  You can measure performance using kubemark,
scheduler_perf, go benchmark tests, or e2e tests on a real cluster with metric
plots.

Examples of how NOT to suggest a performance bug (these lead to a long review
process and waste cycles):

- We *should* be doing X instead of Y because it *might* lead to better
  performance.
- Doing X instead of Y would reduce calls to Z.

The above statements have no value to a reviewer because neither is backed by
data. Writing issues like this lands your PR in a no-man's-land and waste your
reviewers' time.

Examples of possible performance improvements include (remember, you MUST
document the improvement with data):

- Improving a caching implementation.
- Reducing calls to functions which are O(n^2)
- Reducing dependence on API server requests.
- Changing the value of default parameters for processes, or making those values
  'smarter'.
- Parallelizing a calculation that needs to run on a large set of node/pod
  objects.

These issues should always be submitted with (in decreasing order or value):

- A golang Benchmark test.
- A visual depiction of reduced metric load on a cluster (measurable using
  metrics/ endpoints and grafana).
- A hand-instrumented timing test (i.e. adding some logs into the controller
  manager).

Here are some examples of properly submitted performance issues.  If you are new
to kubernetes and thinking about filing a performance optimization, re-read one
or all of these before you get started.

- https://github.com/kubernetes/kubernetes/issues/18266 (apiserver)
- https://github.com/kubernetes/kubernetes/issues/32833 (node)
- https://github.com/kubernetes/kubernetes/issues/31795 (scheduler)

Since performance improvements can be empirically measured, you should follow
the "scientific method" of creating a hypothesis, collecting data, and then
revising your hypothesis.  The above issues do this transparently, using figures
and data rather then conjecture. Notice that the problem is analyzed and a
correct solution is created before a single line of code is reviewed.
