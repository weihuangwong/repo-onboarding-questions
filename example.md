The following is example output when using this skill with Claude.ai.

Prompt:

```
https://github.com/kosukeimai/mediation

I'm being asked to familiarize myself with this package. Can you use the repo onboarding skill to generate a list of questions I should ask the authors.
```

---

# Questions for the `mediation` package authors

Repo: https://github.com/kosukeimai/mediation — v4.5.1, "Causal Mediation Analysis." Maintainer Teppei Yamamoto; seven authors (Tingley, Yamamoto, Hirose, Keele, Imai, Trinh, Wong).

A note on the residue: this package is unusually well-documented for an academic R package. The roxygen blocks carry full estimand definitions and references, there's a JSS vignette (`mediation.Rnw`), a granular `ChangeLog` going back to 2009, and four cited papers anchoring the methods. So most of the usual "why does this exist" provenance is *already recoverable*. The honest residue below is small and concentrated in two places: the test infrastructure, and a few unrecorded build/workflow decisions.

---

## What I could answer from the artifacts (so you don't ask these)

- **What the package implements and the underlying methods** — `DESCRIPTION` cites Imai/Keele/Yamamoto 2010, Imai/Keele/Tingley 2010, Imai/Tingley/Yamamoto 2013, Imai/Yamamoto 2013; estimands defined in `R/mediate.R` roxygen.
- **Dataset provenance** — `R/data.R` has `@source`/`@references` for `framing` (Brader et al. 2008), `jobs` (JOBS II / Vinokur & Schul 1997), `school`/`student` (NCES), and the experimental-design data (Imai/Tingley/Yamamoto 2012). Not a question for a human.
- **Feature evolution and when things landed** — `ChangeLog` is dated and specific (e.g. `mediate_tsls` and the `boot`-package switch in 4.5.0, `speedglm` in 4.5.1).
- **Public API surface** — `NAMESPACE` exports + the S3 method registrations.
- **Why `plot.process` became `plot_process`** — commit `a33f111`: avoiding confusing S3 method registration for CRAN.
- **Why the global-variable declarations exist** — `R/globals.R` explains the `environment(med.fun) <- environment()` pattern feeding intermediates into `med.fun()`.

---

## Open questions for a human

### Validation & trust (highest priority — this is where the real gap is)

1. **The entire test suite is gated on a single developer's username and is therefore dead on CI and for every other maintainer.**
   - *Prompted by*: `tests/testthat/test-mediate.R:4` — `if (isTRUE(unname(Sys.info()["user"])=="weihuang"))`. Everything after that guard is skipped for anyone else.
   - *Risk if wrong*: the package effectively ships **untested** on CRAN's machines and for any new maintainer. Need to know whether this was a deliberate "these are too slow / non-deterministic for CI" decision or just left in. What is the intended way to actually run regression tests?

2. **How was `ref_objects.RData` generated, and is that generation reproducible?**
   - *Prompted by*: the test loads `ref_objects.RData` and compares against it, and comments reference a `gen_ref_objects` script — but **no such script exists anywhere in the git history** (checked across all revs). The reference objects are a committed binary blob with no generator.
   - *Risk if wrong*: the regression baseline is unauditable. If a method changes, no one can regenerate the "known good" outputs without the missing script. Where does `gen_ref_objects` live, and what `mediation` version/seed produced the current blob?

3. **Large blocks of the test file are commented out — abandoned or just slow?**
   - *Prompted by*: the multilevel (`lmer`), group-level treatment, and `ivmediate` test cases at the tail of `test-mediate.R` are all commented out, with `## IV mediate (TODO: implement boot)` sitting over the IV block.
   - *Risk if wrong*: these are exactly the trickier code paths (mixed models, IV). Are they commented because they're broken, non-deterministic, or just slow — and is the `boot` TODO for `ivmediate` still open?

### Decisions without recorded rationale

4. **`speedglm` is a fragile dependency that's already broken once — what's the support intent?**
   - *Prompted by*: `R/use_speed.R` wraps `speedglm::speedglm`; the `use_speed` option exists across `mediate`/`mediate_tsls`; `ChangeLog` 4.5.1 is "Bug fix for speedglm." `speedglm` was archived/orphaned on CRAN for stretches.
   - *Risk if wrong*: if `speedglm` goes away, the `use_speed = TRUE` path dies. Is keeping it a hard requirement, or is it a candidate for removal / replacement with `fastglm` or base `glm.fit`?

5. **Why the 2013-era split of packages between `Depends` and `Imports`, and is `Depends: MASS, Matrix, mvtnorm, sandwich` still load-time-necessary?**
   - *Prompted by*: `DESCRIPTION` puts those four in `Depends` (attaching them for the user) while `lme4`, `Hmisc`, etc. are `Imports`. `ChangeLog` 4.4.3 notes "Moved some packages from Depends to Imports" — a partial migration.
   - *Risk if wrong*: anything still in `Depends` pollutes the user's search path and is harder to drop later. Is the remaining `Depends` set load-bearing or just historical?

### Environment & external dependencies

6. **What's the release/check workflow? There's no CI config in the repo.**
   - *Prompted by*: no `.github/workflows`, no `.travis.yml`, no `appveyor`. Combined with the username-gated tests, there appears to be no automated check of any kind. Recent commits are all "CRAN submission"/"fixing CRAN CHECK issues" done by hand.
   - *Risk if wrong*: a new maintainer won't know what `R CMD check` configuration, which R versions, or which suggested-package matrix the release is actually validated against before going to CRAN.

7. **The `URL` points to a personal project page, not the repo or a pkgdown site — is it canonical?**
   - *Prompted by*: `DESCRIPTION` `URL: https://imai.princeton.edu/projects/mechanisms.html`. Imai is now at Harvard (his email in the same file is `imai@harvard.edu`), so the Princeton URL may be stale.
   - *Risk if wrong*: minor, but it's the documented canonical reference and may 404.

### Provenance & authority

8. **Two vignette sources coexist — `mediation.Rnw` vs `mediation-old.Rnw` — which is authoritative?**
   - *Prompted by*: `vignettes/` contains both, plus two chart JPGs (`mediation-chart.jpg` and `mediation-chart-new.jpg`) and multiple `.bib` files. `ChangeLog` 4.4.1 says the vignette was "updated to the JSS paper."
   - *Risk if wrong*: a maintainer editing docs could update the dead one. Confirm `mediation.Rnw` + `mediation-chart-new.jpg` is the live pair and the `-old` files are kept only for archival.

---

*Categories with no artifact-anchored question (business/stakeholder scope, the "graveyard" beyond what's noted in #3) yielded nothing specific — that means I found no prompt for a question there, not that they're necessarily clean. For an open-source methods package the stakeholder/scope category mostly doesn't apply; the papers are the spec.*
