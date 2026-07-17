# Welcome to bagofseeds 👋

This page contains reusable python utilities.

##  bagof

`bagof` is a namespace package that houses miscelaneous tools that I found useful when implementing python packages.

The core idea behind this project is that each "bag" can be installed on its own, and with a minimal set of dependencies:
- none when possible;
- other bags when useful;
- well-known external packages only if necessary.

📖 **[Documentation](https://bagofseeds.github.io/bagof/)**

## fiery

`fiery` is a namespace package that houses PyTorch utilities and
extensions that I found useful when implementing deep-learning and
medical-imaging code.

The core idea behind this project is that each "match" can be installed on
its own, with a minimal set of dependencies (`torch`, other `fiery` matches
when useful, and well-known external packages only when necessary). Each
match imports under the shared `fiery.` namespace.

📖 **[Documentation](https://bagofseeds.github.io/fiery/)**
