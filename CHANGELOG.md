# Changelog

## [Unreleased]

### Added

- Creating an SCS project also creates a WebProtégé ontology project. The project-page WebProtégé card deep-links into it; members receive EDIT access. Configure the API key in SCS Manager settings (not stored in git). Backfill existing projects with `drush soda_scs_manager:backfill-webprotege-projects --all` (same command on scs-prime).
- Application cards: hover (or tap on mobile) the health icon to start, stop, or restart apps that have a dedicated container (WissKI, MariaDB, triplestore, Jupyter notebook). Shared services (Nextcloud, WebProtégé, JupyterHub) are unchanged.

### Changed

- Removed the **Restart Jupyter** button from the JupyterHub card; notebook stop/restart is on the health icon.

### Fixed

- Mount corrected Nginx `drupal.conf` for SCS Manager so Drupal generates clean URLs without `/index.php` (replaces legacy `?q=` rewrite).
