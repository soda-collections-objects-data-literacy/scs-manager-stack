# Changelog

## [Unreleased]

## [1.1.0] - 2026-11-11

Stack and compose changes since `1.0.0`. Module UI features that were
mistakenly listed under Unreleased live in `soda_scs_manager` instead.

### Added

- Named volume `scs-manager--drupal-config-sync` mounts Drupal's config sync
  directory so `config/sync` survives container recreation.
- Nginx `configs/nginx/drupal.conf` is shipped and mounted so Drupal generates
  clean URLs without `/index.php` (replaces the legacy `?q=` rewrite).
- Varnish `configs/varnish/default.vcl` with backend first-byte and idle
  timeouts of 600 s for long WissKI views.
- PHP-FPM `configs/php-fpm/zz-docker.conf`; the entrypoint points at
  `settings.php`.

### Changed

- Compose service names use three-segment DNS (`manager--drupal--app`,
  `manager--redis--cache`, …) to match the rest of the SCS stack.
- Modules and config mounts are tightened; snapshot host path corrected;
  example env uses hostname variables without a default DB password and pins
  `DRUPAL_IMAGE` to `11.3-php8.4-fpm-bookworm`.
- Xdebug debug overlay logs to `/var/log/xdebug`, keeps OPcache enabled by
  default, and uses lower variable-serialization limits.

### Fixed

- Mounted Nginx config so clean URLs work without `/index.php`.

## [1.0.0] - 2026-01-15

### Added

- Initial tagged release of the SCS Manager stack.
