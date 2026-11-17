# AGENTS.md

Guidance for AI coding agents working in this repository.

## Project overview

A single Helm chart, `rolldice-helm`, that deploys the rolldice app (`ghcr.io/jerrytfleung/rolldice`) to Kubernetes. There is no application source code here, only the chart and its CI.

```
charts/rolldice-helm/
├── Chart.yaml            # chart metadata: version, appVersion, maintainers
├── values.yaml           # default values (documented inline)
└── templates/
    ├── _helpers.tpl      # name/label/serviceAccount helpers (rolldice-helm.*)
    ├── deployment.yaml
    ├── service.yaml
    ├── serviceaccount.yaml
    ├── ingress.yaml      # gated by .Values.ingress.enabled
    ├── httproute.yaml    # Gateway API, gated by .Values.httpRoute.enabled
    ├── hpa.yaml          # gated by .Values.autoscaling.enabled
    ├── NOTES.txt         # post-install instructions
    └── tests/test-connection.yaml  # `helm test` hook (busybox wget)
.github/workflows/
├── lint-test.yml         # PRs to main: ct lint + ct install on kind
└── release.yml           # push to main: chart-releaser → gh-pages
```

## Commands

```bash
helm lint charts/rolldice-helm
helm template my-rolldice charts/rolldice-helm
helm template my-rolldice charts/rolldice-helm --set ingress.enabled=true --set httpRoute.enabled=true --set autoscaling.enabled=true

# Same checks CI runs (requires chart-testing `ct`, and a cluster for install)
ct lint    --chart-dirs charts --target-branch main
ct install --chart-dirs charts --target-branch main
```

After changing templates, render with the optional features on (ingress, httpRoute, autoscaling, serviceAccount.create=false) as well as off. Otherwise you never exercise the gated templates.

## Rules that matter

- **Bump `version` in `charts/rolldice-helm/Chart.yaml` on every chart change.** `ct lint` fails a PR that changes the chart without a version bump, and chart-releaser only publishes new versions. Use SemVer: patch for fixes, minor for new values or features, major for breaking value changes.
- **Keep `appVersion` (Chart.yaml) and `image.tag` (values.yaml) in sync** when you upgrade the app image. The tag falls back to `appVersion` when empty.
- **Update `README.md` along with the chart**: the version table and the Configuration parameter table must match `Chart.yaml` and `values.yaml`.
- **Document new values inline in `values.yaml`** in the existing comment style, and give them defaults that keep `helm lint` and `ct install` passing with no overrides.
- Container port comes from `service.targetPort` and is named `http`. Probes and the Service reference the port by name, so keep that name.
- Use the `rolldice-helm.*` helpers (`fullname`, `labels`, `selectorLabels`, `serviceAccountName`) in new templates instead of hand-building names or labels. Use `{{- with ... }}` / `toYaml | nindent N` as the existing templates do.
- Do not change `selectorLabels`. Deployment selectors are immutable, so a change breaks `helm upgrade` for existing installs.
- Don't edit the `gh-pages` branch by hand; the release workflow manages it.

## Git / PRs

- `main` is the default branch. Work on a feature branch and open a PR to `main`; PRs to `main` run the lint/test workflow.
- Merging to `main` triggers a release, so the PR must include the version bump.
- `.DS_Store` and `.idea/` are gitignored; don't commit editor or OS files.
