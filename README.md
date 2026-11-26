# Cache Poisoning

This lab demonstrates untrusted pull request content crossing into a scheduled
trusted workflow through a shared GitHub Actions cache.

## Vulnerability

The student comments `/build` on a pull request. `Build Untrusted PR Cache`
checks out that pull request and packages its `app/server.js` as build output.

GitHub normally gives low-trust `issue_comment` workflows read-only cache
access. This lab intentionally overrides that protection with:

```yaml
cache-mode: write-only
```

The untrusted workflow can therefore save attacker-controlled content in the
default branch's cache scope. Every attempt uses a unique cache key beginning
with `poisoned-build-` because GitHub cache entries are immutable.

`Deploy Trusted Cache` runs approximately every five minutes. It restores the
newest cache matching `poisoned-build-` and executes `dist/app.js` with the
repository `FLAG` secret in its environment. Scheduled runs report a no-op and
remain successful until a matching poisoned cache exists.

## Setup

1. Create a repository secret named `FLAG`.
2. Allow students to fork the private repository and open pull requests.
3. Allow the student account to comment on its pull request.
4. Use a separate repository and flag per student. In a shared repository, the
   most recently created poisoned cache wins.
5. Scheduled workflows must remain enabled. GitHub can delay scheduled runs, so
   the five-minute cron interval is not an exact execution guarantee.

`workflow_dispatch` is also enabled for instructor testing. Students should not
have repository write access and therefore cannot invoke it directly.

## Exercise

Fork the repository and identify how pull request-controlled build output can be
written to cache state later consumed by the scheduled trusted workflow. Poison
the shared state and recover the flag without modifying the trusted workflow.

## Reset

- Delete the `poisoned-build-*` caches before reusing the repository.
- Rotate `FLAG` between students.
- Disable the scheduled workflow when the lab is not in use to avoid repeated
  executions and unnecessary Actions usage.

## Defenses

- Do not grant write-capable cache access to low-trust workflow triggers.
- Do not package attacker-controlled code in a trusted cache scope.
- Do not use broad restore prefixes across trust boundaries.
- Rebuild trusted output from reviewed source instead of executing cached code.
- Treat all restored caches as untrusted unless their producer is trusted.
