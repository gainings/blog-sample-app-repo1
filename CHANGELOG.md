# Changelog

## [v2026.0921.1](https://github.com/gainings/blog-sample-app-repo1/compare/v2026.0921.0...v2026.0921.1) - 2026-09-21

- Request releases via the release repo's Propose release workflow by @gainings in https://github.com/gainings/blog-sample-app-repo1/pull/18
- Convert release-tag.yml to the request-release action by @gainings in https://github.com/gainings/blog-sample-app-repo1/pull/19
- Use separate GitHub Apps for tagpr and for release requests by @gainings in https://github.com/gainings/blog-sample-app-repo1/pull/20
- Check installation targets instead of repository permissions by @gainings in https://github.com/gainings/blog-sample-app-repo1/pull/21

## [v2026.0921.0](https://github.com/gainings/blog-sample-app-repo1/compare/v2026.0920.4...v2026.0921.0) - 2026-09-21

- Point at ecs/blog-sample-app in the release repo by @gainings in https://github.com/gainings/blog-sample-app-repo1/pull/13
- Follow the service-first layout of the release repo by @gainings in https://github.com/gainings/blog-sample-app-repo1/pull/14
- Use the repository name as the release directory name by @gainings in https://github.com/gainings/blog-sample-app-repo1/pull/15
- Name the release directory by APP_NAME by @gainings in https://github.com/gainings/blog-sample-app-repo1/pull/16
- Tweak hello message by @gainings in https://github.com/gainings/blog-sample-app-repo1/pull/17

## [v2026.0920.4](https://github.com/gainings/blog-sample-app-repo1/compare/v2026.0920.3...v2026.0920.4) - 2026-09-20

- Split tag handling out of tagpr.yml into release-tag.yml (on: push: tags) by @gainings in https://github.com/gainings/blog-sample-app-repo1/pull/10

## [v2026.0920.3](https://github.com/gainings/blog-sample-app-repo1/compare/v2026.0920.2...v2026.0920.3) - 2026-09-20

- Open stg and prd release PRs at the same time; drop the wait-for-release logic by @gainings in https://github.com/gainings/blog-sample-app-repo1/pull/8

## [v2026.0920.2](https://github.com/gainings/blog-sample-app-repo1/compare/v2026.0920.1...v2026.0920.2) - 2026-09-20

- dev on main merge; stg (auto) and prd (manual) on tag by @gainings in https://github.com/gainings/blog-sample-app-repo1/pull/6

## [v2026.0920.1](https://github.com/gainings/blog-sample-app-repo1/compare/v2026.0920.0...v2026.0920.1) - 2026-09-20

- Add nginx sidecar image; release dev and stg via separate PRs by @gainings in https://github.com/gainings/blog-sample-app-repo1/pull/3
- Grant contents:read to jobs that check out the app repo by @gainings in https://github.com/gainings/blog-sample-app-repo1/pull/4

## [v2026.0920.0](https://github.com/gainings/blog-sample-app-repo1/commits/v2026.0920.0) - 2026-09-20

- Build-only mode when AWS_BUILD_ROLE_ARN is unset; tweak hello message by @gainings in https://github.com/gainings/blog-sample-app-repo1/pull/1
