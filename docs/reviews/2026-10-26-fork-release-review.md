# KaronLabs/rea release publication review

Observed on 2026-10-26. Decision: do not merge release PR #1 or publish packages.

## Snapshot and scope

- Fork main: `4a4bbab3c337701febeb6b532ace49041c4876e0`.
- Release PR #1: open, HEAD `24b79b17ee853644344835be476d3bd34cdc1192`.
- Seven files change: manifest, CHANGELOG, installation docs, package.json,
  package-lock.json, server.json and generated package metadata. The PR bumps
  6.7.0 to 6.8.0; it does not itself introduce runtime implementation changes.
- Its changelog includes compact MCP input advertisements, nullable browser
  event fields, environment derived-state reads and replay-only writes,
  reconstruction authority keys, native binding changes and other migrations.
  These are compatibility changes despite the minor increment; this repository
  deliberately uses minor releases with explicit breaking-change notes.
- The generated notes duplicate at least the environment breaking-change entry.
  This review does not certify every accumulated runtime change.

## Verified execution paths

`release.yml` runs on main push, main PR closure and manual dispatch.
Main push updates the release proposal and skips GitHub Release creation.
A merged same-repository `release-please--branches--main--` PR enters
`merged-release`: exact source/checkpoint and finalized-note checks precede
Release Please's GitHub Release creation. CI qualification and clean-runner
artifact rehearsal follow release creation, then npm publication and npm
verification, then MCP Registry publication. Consequently a GitHub Release
can exist even if later qualification or package publication fails.
Manual `phase=publish` provides another route through a frozen release branch.

The successful fork Release run [39718991832](https://github.com/KaronLabs/rea/actions/runs/39718991832)
only executed proposal creation; publication and qualification jobs were skipped.
The PR title [run](https://github.com/KaronLabs/rea/actions/runs/39778524381)
is `action_required`. The PR HEAD check-runs API returned zero check runs.
Successful main CI at another SHA does not validate the release candidate.

## Existing publications and authority

Public npm metadata reports `rea-agents@6.8.0` as `latest`, with maintainer
`morluto`. The npm provenance payload identifies `https://github.com/morluto/rea`,
`.github/workflows/release.yml`, source commit
`160d821ac9a0310afcb8d1fc427862dd6508add5` and
[upstream run 40177214205](https://github.com/morluto/rea/actions/runs/40177214205).
This is a read of the public attestation payload, not independent cryptographic
signature verification. The [upstream release](https://github.com/morluto/rea/releases/tag/rea-agents-6.8.0)
was published 2026-10-25T08:36:02Z. MCP Registry reports the same identity/version
active, published 2026-10-25T09:11:11.166659Z.

The workflow targets upstream's unscoped npm package and MCP namespace.
`id-token: write` permits requesting an OIDC token; it is not registry ownership.
Actual npm trusted-publisher configuration requires authenticated package-level
access and was not available through the connected GitHub tools. No secret
values or publication credentials were requested, printed, or exercised.
Public maintainer information does not prove this fork has zero delegated rights.

[Official npm documentation](https://docs.npmjs.com/trusted-publishers/) requires
matching owner/repository/workflow identity. [Official MCP guidance](https://github.com/modelcontextprotocol/registry/blob/main/docs/modelcontextprotocol-io/github-actions.mdx)
derives GitHub OIDC namespace from repository ownership. The ordinary KaronLabs
OIDC identity therefore does not establish authority over `io.github.morluto/*`.
Retaining upstream metadata in a contribution fork is reasonable; publishing
under it without confirmed delegation and intent is not.

A different artifact cannot safely reuse the existing npm 6.8.0 coordinate.
Existing `scripts/ci/npm-release.mjs` checks artifact integrity and tag selection
before accepting a published version; it refuses mismatches. MCP registry-state
checks likewise refuse differing existing metadata. These checks reduce collision
risk but do not substitute for source/namespace authorization.

## Minimal safeguard

Restrict all seven release workflow jobs to the exact canonical repository
`morluto/rea` with `github.event.repository.fork == false`. This blocks automatic
proposal generation, merged-PR publication, manual preparation/publication,
qualification, rehearsal, npm and MCP jobs on the fork, including jobs using
`always()`. No opt-in variable or credential-based bypass is introduced.
No package identity is renamed and existing integrity checks remain unchanged.

This is a draft-branch change. It does not protect main until reviewed and merged.
Previously started runs, old workflow revisions and direct human publication are
outside this YAML guard. Do not rerun historical publication workflows.
A later deliberate fork release needs an owned npm name and MCP namespace,
consistent metadata and scripts (some hard-code `rea-agents`), registry authority,
version policy and separately reviewed enablement.

## Executed validation

- Regression failed on original workflow: canonical repository guard absent.
- Focused Vitest run: 5 files, 93 tests passed (workflow, checkpoint, recovery,
  release configuration and action pinning). Fork push/PR/manual contexts are
  denied even with all downstream prerequisite results set to success.
- `oxlint tests/boundary/filesystem/releaseWorkflow.test.ts`: exit 0.
- `oxfmt --check` on modified workflow/test: passed.
- `git diff --check`: passed.
- `npm run metadata:check` and `npm run mcp-registry:check`: passed on main
  metadata (6.7.0), not evidence of the PR candidate runtime.
- Exact PR HEAD checkpoint invocation failed: baseline
  `refs/tags/rea-agents-6.7.0` unavailable in the fork. Remote tag lookup also
  returned no matching ref. No upstream tag was injected to mask this failure.
- An initial ESLint invocation was inapplicable (repository uses oxlint and has
  no eslint.config); repository-native lint was then used successfully.

Full application/OS/provider tests, release artifact rehearsal, registry write
permissions and GitHub-hosted execution of the new guard remain unverified.
No release PR was merged, no GitHub Release was created and no package was
published during this review.
