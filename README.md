# STO-ns

**STO-ns** is a lightweight, high-performance, pure Rust mathematical kernel for the **exact** evaluation of two-center Coulomb integrals over $ns$ Slater-Type Orbitals (STOs).

It is designed to be the foundational primitive for quantum chemistry software, specifically optimized for semi-empirical methods (like QEq, EEM, ReaxFF) and ab initio calculations involving spherically symmetric orbitals.

## Features

- **🚀 High Performance**:

  - **Zero Heap Allocation**: All computations happen on the stack.
  - **Algorithmic Optimization**: Uses recurrence relations (**O(N)**) instead of naive summation (**O(N^2)**) for auxiliary integrals.
  - **Compile-Time Precomputation**: Factorials and constants are computed at compile time.
  - **Horner's Method**: Polynomial evaluations are optimized for minimal CPU cycles.

- **🎯 Exact & Stable**:

  - Uses analytical expansions in ellipsoidal coordinates.
  - Automatically handles numerical singularities at $R \to 0$ (one-center limit) and $R \to \infty$.
  - Correctly handles small $\zeta$ differences using Taylor expansions to avoid precision loss.

- **🛰️ Embeddable & Portable**:

  - **`no_std` Support**: Can be used in embedded devices, kernels, or WASM environments.
  - **Pure Rust**: No C/C++ bindings or complex build chains.

## Installation

Add this to your `Cargo.toml`:

```toml
[dependencies]
sto-ns = "0.1.2"
```

To use in a `no_std` environment, disable default features and enable `libm`:

```toml
[dependencies]
sto-ns = { version = "0.1.2", default-features = false, features = ["libm"] }
```

## Usage

### 1. Structural API (Recommended)

The `NsOrbital` struct provides a semantic and safe way to define orbitals and calculate interactions.

```rust
use sto_ns::NsOrbital;

fn main() {
    // Define orbital A: 2s orbital with exponent zeta=1.5
    let orb_a = NsOrbital::new(2, 1.5).expect("Invalid parameters");

    // Define orbital B: 1s orbital with exponent zeta=1.0
    let orb_b = NsOrbital::new(1, 1.0).expect("Invalid parameters");

    // Calculate the Coulomb repulsion integral J(A,B) at distance R=2.0 Bohr
    let r_bohr = 2.0;
    let repulsion = orb_a.repulsion(&orb_b, r_bohr);

    println!("J(2s, 1s) @ R={} is {:.6} Hartree", r_bohr, repulsion);
}
```

### 2. Direct Functional API

For tight loops or internal integration where struct overhead is unwanted, use the kernel function directly.

```rust
use sto_ns::sto_coulomb_integral;

fn main() {
    let r = 2.0;
    // Calculate J(n_a=2, zeta_a=1.5, n_b=1, zeta_b=1.0)
    let result = sto_coulomb_integral(r, 2, 1.5, 1, 1.0);

    println!("Result: {}", result);
}
```

## Mathematical Background

The library evaluates the Coulomb integral:

$$ J\_{AB} = \iint \frac{|\phi_A(\mathbf{r}\_1)|^2 |\phi_B(\mathbf{r}\_2)|^2}{|\mathbf{r}\_1 - \mathbf{r}\_2|} d\mathbf{r}\_1 d\mathbf{r}\_2 $$

Where $\phi$ are normalized Slater-type orbitals:

$$ \phi*{n, \zeta}(r) = N r^{n-1} e^{-\zeta r} Y*{00} $$

The evaluation uses the expansion of the potential in ellipsoidal coordinates, reducing the 6D integral to a sum of auxiliary integrals which are computed via stable recurrence relations.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
