Travis CI
=========

Support for using Travis CI has been added in order to automate continuous integration and pull-request testing.
See [travis-ci.com](https://travis-ci.com/) for more info.

Features:

- Declarative configuration in `.travis.yml`.
- Build matrix testing multiple toolchains, platforms, and sanitizers.
- Dependency caching via the [depends](/depends) system to mirror production and Gitian release environments.
- Automated unit testing (`src/test/test_megacoin`) and functional test suite (`test/functional/test_runner.py`).

For details of the build descriptor, see `.travis.yml` in the repository root.
