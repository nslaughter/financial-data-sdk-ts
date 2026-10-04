# Financial data SDK for TypeScript

A TypeScript SDK demonstration for financial research: retrieve the data
available at a chosen cutoff, resume an interrupted download, and keep the
versions an earlier analysis used.

I'm [Nathan Slaughter](https://nathanslaughter.com/), a software engineer with a
background in investment research and portfolio management. I work mainly in Go
and Python. TypeScript is a common language for API clients, so this SDK
completes the set, checked against the same contract tests as the other two.

**Status:** Project brief. This repository currently contains this README.
The SDK, tests, and runnable examples are planned; nothing described here has
been implemented or tested yet. The demo API and the shared data contract live
in [financial-data-api](https://github.com/nslaughter/financial-data-api). The
dataset is synthetic, and this is a demonstration project, not client work.

## What this project demonstrates

This is a supporting example for my SDK development and maintenance work. It
covers the same customer workflow as the
[Python SDK](https://github.com/nslaughter/financial-data-sdk-python) and the
[Go SDK](https://github.com/nslaughter/financial-data-sdk-go), written the way
a TypeScript developer expects. It is meant to show:

- **One contract, idiomatic in each language.** The three SDKs return the same
  records for the same queries. Each follows its own language's conventions
  for errors, iteration, cancellation, and packaging.
- **The same checks for every SDK.** CI runs the shared data contract's
  expected results against the same pinned demo API image the other SDKs use.
  Those results were written before any client code, so they check this client
  independently of how it was built.
- **Data whose meaning survives the client.** Records keep observation and
  revision identities, observation periods, availability times, decimal
  values, units, and missing values.
- **Downloads that can resume.** Pages and continuation tokens are available
  alongside a record iterator, so a scheduled job can checkpoint its work.
- **Failures a customer can act on.** Denied access, invalid credentials,
  invalid input, throttling, and transient errors are distinguishable from one
  another and from an empty result.

This SDK follows the Python SDK in the first of four stages of a demonstration
for financial data providers. The [financial-data-api](https://github.com/nslaughter/financial-data-api)
supplies the demo API and expands it in the second stage, and the
[financial-data-api-monitor](https://github.com/nslaughter/financial-data-api-monitor)
checks what customers receive from it.

## A successful download should leave the researcher able to explain the result

The example follows a fictional monthly activity index. Its August 2026 value
is released as 102.4 on September 3, then revised to 102.1 on September 10.
A researcher reproducing an analysis made on September 4 still needs 102.4.
A scheduled job that loses its connection on the fourth page needs to know
which records it saved and where to resume.

## How the first demonstration will work

1. Install the packed release in a clean Node.js project and start the demo API
   from the container image that financial-data-api publishes, at a pinned
   version. The demonstration needs no external data credentials.
2. Retrieve the observations available at a chosen cutoff, retaining their
   observation and revision identities.
3. Interrupt a paginated download, resume from its saved checkpoint, and
   compare the local records with the expected results in the shared data
   contract.
4. Follow releases, revisions, and withdrawals through an update cursor,
   keeping the version with the highest revision number current.
5. Revoke the credential's access to the series, force throttling beyond the
   retry budget, and introduce a revision during a download.

The intended interface looks like this. It is a sketch; no package has been
published.

```ts
const client = new Client({ apiKey: process.env.FINANCIAL_DATA_API_KEY! });

for await (const observation of client.observations.iterate({
  seriesId: "activity-index",
  periodStart: "2026-08-01",
  periodEnd: "2026-09-01",
  availableAsOf: "2026-09-04T00:00:00Z",
})) {
  console.log(observation.periodStart, observation.value, observation.revisionId);
}
```

## Design choices the examples will make visible

- **Promises and async iteration without hidden work.** Methods return
  promises. `iterate` returns an async iterator that fetches a page only when
  the loop asks for one, and leaving the loop stops further requests. A
  page-level method returns each page with its continuation token for jobs
  that checkpoint.
- **Cancellation with `AbortSignal`.** Every method accepts a signal, and a
  deadline such as `AbortSignal.timeout()` covers requests and retry waits.
- **Values that JavaScript numbers can't distort.** JavaScript numbers are
  binary floating point, so `value` stays a decimal string. Observation periods
  stay `YYYY-MM-DD` strings rather than `Date` objects, which would attach a
  time zone.
- **Types that describe the data.** Strict TypeScript types use the language's
  camelCase names, mapped from the API's field names. A discriminated union on
  `changeType` makes a withdrawal's missing value explicit in the type.
- **Errors that `instanceof` can tell apart.** Error classes identify denied
  access, invalid credentials, throttling, and an expired snapshot. Each carries
  the provider's request ID and any `Retry-After`, never the credential. An
  empty result is an iteration with no records.
- **The client fits the application around it.** It uses the runtime's
  built-in `fetch`, which customers can replace, and has no runtime
  dependencies. It targets server-side runtimes, because an API key doesn't
  belong in a browser.

## Scope of the first release

The first release covers the same synthetic dataset and research workflow as
the Python SDK: authentication, typed records, pagination, errors, bounded
retries, resumable downloads, and update following. Whether the later migration
stage covers this SDK will be decided when that stage is planned.

## The demonstration is complete when

- The packed release installs in a clean project, type-checks, and the
  documented workflow runs against the pinned demo API, passing the shared
  contract's checks for this stage.
- Authentication and rate-limit errors are reported distinctly, and retries
  stop within their configured bounds.
- An interrupted paginated download resumes with no missing or duplicate
  records.
- The September 4 query still returns 102.4 after the revision is loaded.

## What the repository will contain

- A customer-focused README and quickstart.
- A runnable research example, and a scheduled-job example that checkpoints
  and resumes.
- Type declarations and reference documentation mapped to the shared data
  contract.
- CI that packs the package, installs it in a clean project, and runs the
  type check, tests, examples, and the contract's checks against the pinned
  demo API image on each supported Node.js LTS release.
- A tagged release with the packed package attached.
- Documented limitations and a clear demonstration label.

## Related projects and writing

- [financial-data-sdk-python](https://github.com/nslaughter/financial-data-sdk-python)
  and [financial-data-sdk-go](https://github.com/nslaughter/financial-data-sdk-go):
  the same client in Python and Go.
- [financial-data-api](https://github.com/nslaughter/financial-data-api): the
  demo API, the shared data contract, and the full API.
- *Building an SDK your customers love*: an article on the design behind these
  SDKs, in preparation. I'll link it here when it is published.

## Work with me on an SDK your customers can use

I take on SDK projects scoped around the workflows your customers need to
complete. The work can include interface design, implementation, documentation,
release packaging, compatibility checks, and ongoing maintenance.

[Discuss an SDK project](https://www.linkedin.com/in/nathan-slaughter) with the
API, target language, and customer workflow you need to support.
