# Xamarin Apple documentation migration

This contributor planning directory contains the initial per-guide porting plan for native Apple documentation. It is outside `docs/` and isn't published to Microsoft Learn.

The inventory was assessed on October 29, 2026. It is **static, rule-assisted editorial triage**, not a completed runtime/API audit or a claim that the archived examples work on .NET. Review each article's decision and prerequisites when starting its port.

| File                                      | Purpose                                                                        |
|-------------------------------------------|--------------------------------------------------------------------------------|
| `xamarin-apple-porting-plan.md`           | Scope, sequencing, acceptance criteria, sample coordination, per-file plans    |
| `xamarin-apple-guide-inventory.csv`       | 591 archived Apple/shared/tooling files, sorted by priority, with dependencies |
| `macios-maintained-guide-inventory.csv`   | 35 current SDK-maintained guides and proposed publication/retention decisions  |
| `excluded-source-files.csv`               | 884 archived Markdown/YAML files outside the native Apple scope                |
| `inventory-summary.json`                  | Source revision, scope, and inventory counts                                   |

The archived source is pinned to [MicrosoftDocs/xamarin-docs@0506e3bf14b5](https://github.com/MicrosoftDocs/xamarin-docs/tree/0506e3bf14b520776fc7d33781f89069bbc57138). Each selected source file has one inventory row with its hash, proposed action, destination, major work, and external prerequisites. The 591 selected files comprise 581 Markdown articles/includes and 10 navigation/landing YAML files. They and the 884 exclusions account for the 1,475-file Markdown/YAML source corpus.

## Order of work

| Priority   | Files   | Work                                                                     |
|------------|---------|--------------------------------------------------------------------------|
| P0         | 35      | Replace onboarding and landing material using native SDK templates       |
| P1         | 217     | Documentation-only ports and merges; no required historical sample port  |
| P2         | 150     | Substantial documentation, runtime, or tooling rewrites                  |
| P3         | 34      | External sample, native-library, hardware, or service verification gates |
| P4         | 155     | Don't port; preserve the archive                                         |

"Drop" means omit from the modern port, **not delete archived documentation**. Existing public URLs must remain until an equivalent page exists and the publishing owner approves targeted redirects.

The first P0 implementation is in [`docs/ios/get-started/`](../../docs/ios/get-started/): native iOS, AppKit macOS, tvOS, and UIKit Mac Catalyst create/build/run tutorials, linked from the existing hub and TOC. Shared prerequisites and simulator guidance are in [`docs/ios/includes/get-started/`](../../docs/ios/includes/get-started/). This doesn't mean all 35 P0 source articles have equivalent modern coverage.

Sample porting remains in the existing [dotnet/macios#20212](https://github.com/dotnet/macios/issues/20212) effort and its [sample inventory](https://gist.github.com/rolfbjarne/d468157fc870fdd9a8fd178ee67a786a). No samples are ported here and no second sample queue is created. Sample mappings describe that inventory's historical state, not live port completion. A related sample link alone shouldn't block a self-contained documentation port; sample-driven walkthroughs need their explicit gates resolved.

## Verification boundaries

Native iOS simulator, AppKit, and Mac Catalyst template builds were verified with SDK 10.0.301 and Apple workloads 26.2.10233. AppKit and Mac Catalyst also built and launched on the Intel host with their x64 runtime identifiers. The arm64 builds don't establish execution on Apple silicon.

iOS simulator launch, tvOS build/run, interactive UI behavior, physical devices, signing/distribution, and a full Learn publishing render remain unverified. The tvOS tutorial was checked against the released .NET 10 SDK template and run-target sources; the tvOS workload wasn't installed solely for documentation validation. Workload installation commands are documented, not exercised by changing this shared machine's workload state.

The full inventory isn't a runtime-tested migration. Its priorities and destinations are proposed editorial decisions, and the published pages and incoming navigation must be verified before closing the onboarding gap tracked by [dotnet/macios#17344](https://github.com/dotnet/macios/issues/17344).
