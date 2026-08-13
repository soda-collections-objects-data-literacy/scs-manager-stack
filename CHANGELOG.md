# Changelog

## [Unreleased]

### Added

- Application cards: hover (or tap on mobile) the health icon to start, stop, or restart apps that have a dedicated container (WissKI, MariaDB, triplestore, Jupyter notebook). Shared services (Nextcloud, WebProtégé, JupyterHub) are unchanged.

### Changed

- Removed the **Restart Jupyter** button from the JupyterHub card; notebook stop/restart is on the health icon.

### Fixed

- Mount corrected Nginx `drupal.conf` for SCS Manager so Drupal generates clean URLs without `/index.php` (replaces legacy `?q=` rewrite).
