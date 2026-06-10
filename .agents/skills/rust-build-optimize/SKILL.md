---
name: rust-build-optimize
version: 1.0.0
description: Add Rust build optimization setting into `Cargo.toml` file
language: en_US

inputs:

output:
  format: toml
  snippet: |
    [profile.release]
    opt-level = "z"
    lto = true
    codegen-units = 1
    panic = "abort"
    strip = true
---

# Rust build optimizatio Skill

## Purpose

Add a snippet to `Cargo.toml` file for current Rust project

## The goal

The optimization setting MUST be applied to project. You are making sure It is and apply if not.

## When to use this skill

Before using this skill make sure this project is written in Rust:

- Check if file `Cargo.toml` exists in the root directory of the project
- `Cargo.toml` file does not have optimization setting yet. If It does, do nothing, but tell the user optimization settings is already applied and show them.

## Optimization setting

There are build optimizations to be applied in the `Cargo.toml` file.
They looks like this:

```yaml
[profile.release]
opt-level = "z"
lto = true
codegen-units = 1
panic = "abort"
strip = true
```

There are `profile.release` section name and keys provided:

- `opt-level`
- `lto`
- `codegen-units`
- `panic`
- `strip`

## Classification rules

### Optimization setting is missing

You MUST add It and tell the user It is applied:

```markdown
Optimization is already applied. Here is a config:
[CONFIG_PLACEHOLDER]
```

Replace [CONFIG_PLACEHOLDER] with the snipped above

### Optimization setting has all the keys listed in the previous unit

Do nothing, but tell the user they are already there by saying:

```markdown
Optimization is already applied. Here is a config:
[CONFIG_PLACEHOLDER_FROM_CARGO_TOML]
```

Replace [CONFIG_PLACEHOLDER_FROM_CARGO_TOML] with the existing config from `Cargo.toml`

### Optimization settings has some keys from the list above, but not all of them

Merge existing setting and the settings you need to use:

- Do not replace existing values with values from snippet above
- Add missing keys and values

Tell the user the optimization is applied and show merged result:

```markdown
Optimization is already applied. Here is a config:
[MERGED_CONFIG]
```

Replace [MERGED_CONFIG] with the existing config from `Cargo.toml` and keys you inserted

## Validation checklist

Before returning the final answer, verify that:

- `Cargo.toml` has correct `TOML` configuration format
- `Cargo.toml` has the configuration supported by actual Rust version (2024 edition)

## Final answer rule

- Return only the requested answer
- Do not explain your reasining
- Do not include alternatives
- Do not include confidence
