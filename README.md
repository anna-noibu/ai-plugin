# Noibu AI Plugin

Official Noibu plugin for AI clients. Access Noibu products direclty from your AI coding tool.

## Description

Built for ecommerce, the Noibu plugin bridges the gap between customer experience and revenue by connecting Claude
directly to your store's session, error, and conversion data through Noibu — and to the marketing, support, and commerce
platforms that put insight into motion. Go beyond analysis: surface what's costing you revenue, take action across your
stack, build workflows to automate work end-to-end.

## Installation

### Claude Desktop (manual)

1. Download the Noibu plugin: https://github.com/Noibu/ai-plugin/releases/latest/download/noibu.zip
2. Go to **Customize**
3. Click **Create plugin** → **Upload plugin**
4. Upload the `noibu.zip` file

### Claude Code

Install directly from the marketplace — no download required:

```shell
/plugin marketplace add Noibu/ai-plugin
/plugin install noibu@noibu-plugins
/reload-plugins
```

## Releases & distribution

This repo (`Noibu/ai-plugin`, public) is the source of truth and feeds two channels:

- **Claude Code** — installs from this repo as a marketplace (`Noibu/ai-plugin`).
- **Cowork** — served from the private mirror `Noibu/ai-plugin-sync`.

Every push to `main` runs `.github/workflows/release.yml`, which:

1. Bumps the patch version (from the latest `v*` git tag),
2. Stamps it into `src/.claude-plugin/plugin.json` **and commits it back to `main`** (`[skip ci]`),
3. Tags + creates a GitHub release with `noibu.zip`,
4. Force-pushes `main` to the private `ai-plugin-sync` mirror.

> **Why the commit-back matters:** clients read the `version` field from `plugin.json`
> at repo `HEAD`, not from the release zip. Claude Code only auto-updates when that
> string changes. If the bump isn't committed, the version stays frozen and **no client
> ever updates** even though content moves. Do not remove the "Commit stamped version"
> step.

To distribute to all Claude Code users automatically, deploy a managed-settings.json
via MDM with `extraKnownMarketplaces` (pointing at `Noibu/ai-plugin`, `autoUpdate: true`)
and `enabledPlugins: { "noibu@noibu-plugins": true }`.

## How to develop

1. Clone and install the plugin
2. Authenticate with Noibu

## Documentation

https://help.noibu.com/articles/7434601323-installing-the-noibu-plugin-for-claude

## Example Use Cases

### Example 1:

**Spot and act on your top revenue opportunities on autopilot**: Every Monday, surface my highest-impact opportunities
across acquisition, products, and experience, and help me take action on each.

### Example 2:

**Optimize your checkout**: Find where shoppers are dropping off, the payment and delivery methods they're using, and
the technical issues hurting conversion, then tackle the top fixes.

### Example 3:

**Resolve technical issues at the source**: What errors and performance issues are costing my store the most revenue,
and help me resolve them.

### Example 4:

**Uncover and resolve shopper friction**: Surface where real shoppers are struggling, through session replays and
revenue heatmaps, and act on the highest friction.

### Example 5:

**Turn product views into sales**: Find which products and collections are getting views but no sales, diagnose
why they aren't converting, and execute on the biggest opportunities.

**Example 6:
Shift marketing spend with site behaviour**: Surface which campaigns are sending high-quality traffic, identify
how to lift upper-funnel conversion, and reallocate budget accordingly.

## License

MIT