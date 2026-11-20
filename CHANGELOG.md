# Changelog

Every release of apogee-container, newest first. Each is a `vX.Y.Z` tag with a GitHub
release carrying its section; the `grav` line in `grav.toml` says which
compiler versions it builds with.

## Unreleased

## 0.2.0 (2026-11-20)

### Requirements

- Needs grav 1.9.0 or later, and uses the 0.2.0 releases of the packages it
  depends on.

## 0.1.0 (2026-11-19)

The first tagged release. Needs grav 1.7.0 or newer.

Dependency injection for [Apogee](https://github.com/venomous-maker): bind what a type means once, then ask for it anywhere. Classes are built by wiring their constructors' parameters, interfaces included.
