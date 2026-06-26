# Shared advisory-acceptance rationale

Both `deny.toml` (`[advisories].ignore`) and `audit.toml` (`[advisories].ignore`)
must carry the SAME two entries with the SAME rationale, kept in sync. The kits
reach both advisories only through the Apache `parquet` decoder they use to read
bundled vendor data; no first-party code in any kit uses `paste` or `thrift`
directly. Mirror the wording below, adapting only the feature-gating sentence to
the kit's actual `parquet` wiring (default-on `parquet-loader` feature vs an
unconditional `parquet` dependency).

```toml
ignore = [
    # paste 1.0.15 — RUSTSEC-2024-0436, unmaintained (no CVE; the crate is
    # feature-complete and the author archived it).
    #
    # Reachability: `paste` reaches this kit ONLY as a transitive proc-macro
    # of `parquet`, which the kit uses to read its bundled vendor data.
    # <FEATURE-GATING SENTENCE: e.g. "parquet is gated behind the default-on
    # parquet-loader feature; disabling it drops the transitive paste at the
    # cost of constructing values from explicit data instead of bundled
    # parquet." OR "parquet is an unconditional dependency of this crate.">
    # No first-party code in this kit uses paste.
    #
    # Release-gate condition: remove this ignore when apache/arrow-rs drops the
    # paste dependency from parquet, or by the review date below, whichever
    # comes first.
    # Tracking: apache/arrow-rs (drop paste dep, long-term).
    # Review date: 2026-11-30.
    "RUSTSEC-2024-0436",

    # thrift 0.17.0 — GHSA-2f9f-gq7v-9h6m, Apache Thrift memory-allocation
    # with-excessive-size-value vulnerability. No fixed thrift crate has been
    # published upstream as of the review date, so there is no version to
    # upgrade to.
    #
    # Reachability: `thrift` ships inside the `parquet` metadata decoder on the
    # same edge as paste above. The kit hits Apache Thrift only via parquet's
    # internal metadata path on a successful read of bundled vendor data; the
    # vulnerable unbounded-deserialisation path is not exercised on the kit's
    # API surface.
    #
    # Release-gate condition: remove this ignore when Apache publishes a fixed
    # thrift crate (and parquet rebumps), or by the review date, whichever
    # comes first.
    # Tracking: apache/thrift, apache/arrow-rs.
    # Review date: 2026-11-30.
    "GHSA-2f9f-gq7v-9h6m",
]
```
