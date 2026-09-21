# petclinic-monitor

A simple uptime monitor for the hosted version of
[Pet Clinic](https://github.com/cgerard321/champlain_petclinic).

Every 5 minutes, GitHub Actions probes each deployed service from GitHub's
cloud runners. Each service has its own workflow, so each one gets its own
live status badge.

## Status


| Service         | Endpoint                                                           | Status                                                                                                                                                                                                                            |
|-----------------|--------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Customer Portal | https://petclinic.benmusicgeek.synology.me/                        | [![Customer Portal Health](https://github.com/DylanBrass/petclinic-monitor/actions/workflows/health-customer-portal.yml/badge.svg)](https://github.com/DylanBrass/petclinic-monitor/actions/workflows/health-customer-portal.yml) |
| Backend API     | https://petclinic-backend.benmusicgeek.synology.me/actuator/health | [![Backend API Health](https://github.com/DylanBrass/petclinic-monitor/actions/workflows/health-backend-api.yml/badge.svg)](https://github.com/DylanBrass/petclinic-monitor/actions/workflows/health-backend-api.yml)             |
| Employee Portal | https://petclinic-emp-portal.benmusicgeek.synology.me/login        | [![Employee Portal Health](https://github.com/DylanBrass/petclinic-monitor/actions/workflows/health-employee-portal.yml/badge.svg)](https://github.com/DylanBrass/petclinic-monitor/actions/workflows/health-employee-portal.yml) |
| Management UI   | https://petclinic-management-ui.benmusicgeek.synology.me/login     | [![Management UI Health](https://github.com/DylanBrass/petclinic-monitor/actions/workflows/health-management-ui.yml/badge.svg)](https://github.com/DylanBrass/petclinic-monitor/actions/workflows/health-management-ui.yml)       |

A green badge means the last check passed. A red badge means it failed.
Click a badge to see the run history and logs for that service.

## How it works

Each workflow in [`.github/workflows/`](.github/workflows/) runs on a
schedule (`*/5 * * * *`) and can also be started by hand with
`workflow_dispatch`.

The check is a single `curl` call:

```bash
curl -L -m 10 -s -o /dev/null -w "%{http_code}" "$URL"
```

- `-L` follows redirects.
- `-m 10` gives the whole request 10 seconds.

A service counts as **up** when curl succeeds and the final HTTP status is
`200`, `301` or `302`. It counts as **down** when curl itself fails
(timeout, DNS error, refused connection, TLS error) or when any other status
comes back. A failed check fails the job, which turns the badge red.

The checks run on GitHub-hosted `ubuntu-latest` runners, outside the network
hosting the services. If the server, its power or its internet connection
goes down, the check fails and the badge turns red.

| Workflow | Service |
| --- | --- |
| `health-customer-portal.yml` | Customer Portal |
| `health-backend-api.yml` | Backend API |
| `health-employee-portal.yml` | Employee Portal |
| `health-management-ui.yml` | Management UI |

## Run a check manually

1. Open the **Actions** tab.
2. Pick a workflow in the sidebar.
3. Click **Run workflow**.

## Add another service

1. Copy one of the workflow files, for example `health-main-portal.yml`, to
   `.github/workflows/health-<name>.yml`.
2. Change the workflow `name:` and the `NAME` and `URL` values in `env:`.
3. Add a row for it in the status table above, pointing the badge at the
   new file name.
4. Push to `main`, then run the new workflow once from the Actions tab.

## Known limitations

- **Timing is best-effort.** GitHub's scheduler can start scheduled runs a
  few minutes late, and 5 minutes is the shortest interval it supports.
- **Badges show the last completed run**, so they can lag behind a change by
  a few minutes. GitHub may also cache badge images in READMEs.
- **Scheduled workflows are paused** after 60 days without activity in the
  repository. If the checks stop, re-enable the workflows from the Actions
  tab or push a small commit.
