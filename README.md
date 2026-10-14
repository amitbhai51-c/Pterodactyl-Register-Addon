# Pterodactyl Register Feature Addon

A Bash installer by **Amitbhai51** for installing the `wemx/pterodactylregister` package on an existing Pterodactyl Panel. The installer runs the package's Artisan installation command, clears Laravel caches, adjusts storage/cache permissions, and restarts Nginx.

> **Important:** This installer changes files inside your Pterodactyl Panel installation. Back up your panel files and database first, and review the script before running it on a production server.

## What the installer does

The supplied installer script performs these actions in order:

1. Checks that `/var/www/pterodactyl` exists.
2. Changes into the Pterodactyl panel directory.
3. Installs `wemx/pterodactylregister` using Composer.
4. Runs `php artisan register:install`.
5. Clears Laravel's optimized, view, configuration, and application caches.
6. Sets permissions on `storage` and `bootstrap/cache` to `755`.
7. Changes ownership of the panel directory to `www-data:www-data`.
8. Restarts Nginx.

The script displays progress messages for the registration installation steps. The actual registration-page changes are performed by the package installer; the Bash script itself does not directly edit the login-page template.

## Requirements

- An existing Pterodactyl Panel installation at `/var/www/pterodactyl`.
- A Linux server with Bash and `sudo` available.
- PHP and the PHP extensions required by your Pterodactyl Panel installation.
- Composer installed and available in `PATH`.
- Nginx managed by systemd (`systemctl`).
- Root or equivalent permissions to change panel file ownership and restart Nginx.
- Network access to download the Composer package.

This script assumes the standard panel directory. If your panel is installed elsewhere, update `PTERO_DIR` in the script before running it.

## Installation

### Option 1: Run a copy of the script

1. Save the supplied Bash installer as `install.sh` on your server.
2. Review the file before executing it.
3. Run:

```bash
chmod +x install.sh
sudo bash install.sh
```

Keep the terminal open until the script finishes. Check the output for errors; the script currently does not stop automatically if every individual installation command fails.

### Option 2: One-command install from GitHub

After you upload `install.sh` to your own GitHub repository, replace the placeholders below with the correct repository owner, repository name, and branch, then run this on the panel server:

```bash
curl -fsSL "https://raw.githubusercontent.com/<GITHUB_USERNAME>/<REPOSITORY>/<BRANCH>/install.sh" -o install.sh && less install.sh && sudo bash install.sh
```

This downloads the script, lets you review it in `less`, and then runs it with elevated permissions. Do not use the command until the placeholders have been replaced with your real GitHub details.

## Verify the installation

Run these commands on the server:

```bash
cd /var/www/pterodactyl
composer show wemx/pterodactylregister
php artisan list | grep -i register
sudo systemctl status nginx --no-pager
```

Then open your panel in a browser and check whether the registration option is available on the login page. The exact behavior and compatibility depend on the installed package version and your Pterodactyl Panel version.

## Manual commands used by the installer

These are the main commands run by the script, shown for reference:

```bash
cd /var/www/pterodactyl
composer require wemx/pterodactylregister
php artisan register:install
php artisan optimize:clear
php artisan view:clear
php artisan config:clear
php artisan cache:clear
chmod -R 755 storage bootstrap/cache
chown -R www-data:www-data /var/www/pterodactyl
sudo systemctl restart nginx
```

Do not run the manual commands separately if the installer has already completed successfully.

## Backup before installing

At minimum, make a copy of the panel directory and a database backup using the method appropriate for your database engine and configuration. A simple files backup is:

```bash
sudo tar -czf "/root/pterodactyl-files-backup-$(date +%F-%H%M%S).tar.gz" -C /var/www pterodactyl
```

This backs up panel files only; **it does not back up your database**. Back up your panel database separately before installing the addon.

## Troubleshooting

### `Pterodactyl directory not found!`

The script expects the panel to be installed at `/var/www/pterodactyl`. Confirm your installation path and update `PTERO_DIR` in `install.sh` if necessary.

### `composer: command not found`

Install Composer for your server's PHP environment, then confirm it is available by running:

```bash
composer --version
```

### `register:install` command not found or fails

Check that Composer successfully installed the package and that you are using the PHP version configured for your Pterodactyl Panel:

```bash
cd /var/www/pterodactyl
composer show wemx/pterodactylregister
php artisan list | grep -i register
```

If the package is incompatible with your panel version, do not keep rerunning the installer; consult the package's documentation and release information.

### Nginx does not restart

Check its status and configuration:

```bash
sudo nginx -t
sudo systemctl status nginx --no-pager
sudo journalctl -u nginx -n 100 --no-pager
```

### Permission problems

The installer changes ownership of the entire panel directory to `www-data:www-data`. This may not match every server's setup. Verify the required ownership for your own installation before rerunning that command.

## Important notes

- The installer does not create a backup or provide an automatic rollback.
- The current script does not use `set -e`, so it can print a success message even if an earlier command failed. Review each command's output and verify the installation afterward.
- The script uses an unpinned Composer requirement (`composer require wemx/pterodactylregister`), so the package version resolved at install time may change.
- Test the addon on a staging panel before installing it on a production panel.
- This README documents the supplied script; it does not guarantee compatibility with every Pterodactyl Panel version.

## Credits

Installer author: **Amitbhai51**

Register package: [`wemx/pterodactylregister`](https://packagist.org/packages/wemx/pterodactylregister)
