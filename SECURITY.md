# Security

## CVE-2025-26519 – musl libc EUC-KR iconv out-of-bounds write

| Field | Value |
|---|---|
| CVE | [CVE-2025-26519](https://www.cve.org/CVERecord?id=CVE-2025-26519) |
| GHSA | [GHSA-xpv5-92cc-8f65](https://github.com/advisories/GHSA-xpv5-92cc-8f65) |
| Affected systems | Linux targets built against **musl libc 0.9.13 – 1.2.5** |
| Fixed in musl | 1.2.6 |

### Summary

musl libc versions 0.9.13 through 1.2.5 contain a heap-based out-of-bounds
write in the `iconv` implementation when converting **EUC-KR** encoded text to
UTF-8.  The write is triggered by byte sequences that are outside the valid
EUC-KR range; a malformed input can overwrite adjacent heap memory, potentially
leading to crashes or arbitrary code execution.

Because `rust-iconv` is a thin wrapper around the system `iconv(3)` C
function, any Rust program that uses this crate on an affected musl system and
processes untrusted EUC-KR data is exposed to the same vulnerability.

### Mitigation in this crate

Starting with the commit that introduced this document, `Iconv::convert()`
validates EUC-KR input on **`musl` targets** (`#[cfg(target_env = "musl")]`)
before forwarding it to the system `iconv`.  Any byte sequence that falls
outside the valid EUC-KR ranges is rejected with `IconvError::InvalidInput`
rather than being passed to the potentially vulnerable C library.  This
prevents the out-of-bounds write from being reached.

The validation accepts:
* Single-byte values `0x00`–`0x7F` (ASCII).
* Two-byte sequences whose lead byte is in `0xA1`–`0xFE` and whose trail byte
  is in `0xA1`–`0xFE` (standard EUC-KR / KS X 1001).

Any other byte is rejected as invalid.

### Recommended action

* **Upgrade musl** to 1.2.6 or later on affected systems.
* If you cannot upgrade musl, update this crate to a version that includes the
  input-validation mitigation described above.
* Avoid passing untrusted EUC-KR data to `iconv` on unpatched musl systems.

### References

* <https://www.cve.org/CVERecord?id=CVE-2025-26519>
* <https://github.com/advisories/GHSA-xpv5-92cc-8f65>
* musl libc fix: <https://git.musl-libc.org/cgit/musl/commit/?id=f3490d8a7ce6c12da0c83e0ad7b05df4dc03cb9e>
