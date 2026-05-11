rust-iconv
==========

[![Build Status](https://travis-ci.org/andelf/rust-iconv.svg?branch=master)](https://travis-ci.org/andelf/rust-iconv)

iconv (libiconv) bindings for Rust.

## Security notice — CVE-2025-26519

musl libc versions **0.9.13 – 1.2.5** contain an out-of-bounds write in
`iconv` when converting **EUC-KR** text to UTF-8 with malformed input
([CVE-2025-26519](https://www.cve.org/CVERecord?id=CVE-2025-26519),
[GHSA-xpv5-92cc-8f65](https://github.com/advisories/GHSA-xpv5-92cc-8f65)).

This crate mitigates the issue on musl targets by validating EUC-KR input
before passing it to the system `iconv`.  Byte sequences outside the valid
EUC-KR range are rejected with `IconvError::InvalidInput` instead of being
forwarded to the vulnerable C library.

**Recommended action:** upgrade musl to 1.2.6 or later.  See
[SECURITY.md](SECURITY.md) for full details.

# Installation

    cargo build

# Testing

    cargo test

# Usage

Refer ``lib.rs``.
