# Changelog

## [Unreleased]

### Added

- `SodaScsDockerDns` resolves shared container DNS from `SCS_CONTAINER_*` / `INTERNAL_TS_BASE` / `NEXTCLOUD_MOUNTER_RC_URL` env (with settings override), so naming migration does not require PHP hardcodes.
- Creating an SCS project also creates a WebProtégé ontology project. The project-page WebProtégé card deep-links into it; members receive EDIT access. Configure the API key in SCS Manager settings (not stored in git). Backfill existing projects with `drush soda_scs_manager:backfill-webprotege-projects --all` (same command on scs-prime).
- Application cards: hover (or tap on mobile) the health icon to start, stop, or restart apps that have a dedicated container (WissKI, MariaDB, triplestore, Jupyter notebook). Shared services (Nextcloud, WebProtégé, JupyterHub) are unchanged.

### Changed

- Central application cards are titled JupyterHub, Nextcloud, and WebProtégé (not the per-user instance name). WissKI, database, and triplestore cards keep the instance title.
- Removed the **Restart Jupyter** button from the JupyterHub card; notebook stop/restart is on the health icon.
- The main-menu **Projects** tab is removed; projects are opened from the dashboard. Project members can be added (owners) and removed (self / admin / owner) on the project page.

### Fixed

- SSO login no longer returns HTTP 500: `WsodCatcherSubscriber` no longer intercepts Drupal's `EnforcedResponseException` (used for the OpenID Connect redirect to Keycloak).
- Mount corrected Nginx `drupal.conf` for SCS Manager so Drupal generates clean URLs without `/index.php` (replaces legacy `?q=` rewrite).
