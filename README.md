# gridpane-wordpress-template

This repo is an example of a repo configured for full deployment (all wp core files, plugins and themes managed by git repo).

As this repo has **none** of the GridPane MU plugins nor integrations plugins, it is appropriate for a site with:

- Nginx/LSCache Page Caching - not enabled
- Redis Object Caching - not enabled
- WPFail2Ban Integration - not enabled
- SendGrid SMTP Integration - not enabled
- Additional Security Features
    - Disable Emoji - not enabled
    - Disable RSS - not enabled
    - Disable Username Enumeration - not enabled
    - Block WP Scan Agent - not enabled
    - Block WP Version - not enabled
- SSO WP Login - not enabled 

We have a different template repo example available for sites where all the above are enabled [here (GridPane-Dev Repo)](https://github.com/GridPane-Dev/gridpane-wordpress-template-all-plugins)

## .gpconfig directory

### Deployment Type

As no `.gpconfig/hybrid` token file exists within this repo, it will be deployed as `GridPane Full Git` type deployment.

This means that the repository will be expected to contain all:
- Core
- Plugins
- Themes
For your WordPress site.

WordPress Core, Plugins and Themes will be owned by root and the system php user will not be able to write to these files.

In addition we will be enabling the `define('DISALLOW_FILE_MODS',true);` constant to disallow file edits.

All updates will need to be done via Git.

### Deployment Scripts

You will find the following empty scripts:
```shell
.gpconfig/predeploy-server.sh
.gpconfig/predeploy.sh
.gpconfig/postdeploy.sh
.gpconfig/postdeploy-server.sh
```

These scripts will execute in the above order.

The `-server.sh` scripts are run as root, with the others running as the site system user.

The scripts are run from the `/.gpconfig` directory.

The root scripts can be useful for running scripts across the server and microservices which need elevated privileges, be careful.

The system user scripts can be useful for running system user functionality, php, wp-cli etc.

Remember to add the correct path to wp core files for wp-cli commands, in the case of a full deploy this would be the directory above.

### Release Directory Retention

```
.gpconfig/keep.releases
```

This repo contains this file, the contents of the file can be an `integer` only, and the integer must be greater than or equal to `3` - this represents the minimum number of releases we will allow you to retain.

This `integer` is the number of releases that the system will retain in the following directory:
```
/var/www/{site.domain}/releases
```

Please see below for details of the releases and release directory naming convention.


## Plugins

This repo only contains:
```shell
wp-content/plugins/akismet
wp-content/plugins/hello.php
wp-content/plugins/index.php
```

These are the base files for a blank WP install.

Based on this, this repo represents a site with:
- Nginx/LScache Page Caching - disabled
- Redis Object Caching - disabled
- WP Fail2Ban integration - disabled

## MU-Plugins

This repo contains no `wp-content/mu-plugins` directory.

Based on this, then it is suitable for a site with the following configurations in GridPane
- Additional Security Settings
    - Disable Emoji - inactive
    - Disable RSS - inactive
    - Disable Username Enumeration - inactive
    - Block WP Scan Agent - inactive
    - Block WP Version - inactive
- Sendgrid SMTP integration- inactive
- SSO login - inactive

## Themes

This repo is configured with base core themes from `twentytwenty` to `twentytwentytwo`

```shell
wp-content/themes/twentytwenty
wp-content/themes/twentytwentyone
wp-content/themes/twentytwentytwo
```

## Directory Structure Changes

### Directory Structure when Git Integration is enabled

When you first enable the Git Integration Connection for a site, we run a function that modifies the Site's directory file structure.
```
root@10alpha:/var/www/test.site# ls -la
total 52
drwxr-xr-x+ 9 test10350 test10350 4096 Jun 29 13:03 .
drwxr-xr-x+ 6 root      root      4096 Jun 29 13:02 ..
drwxr--r--  3 root      root      4096 Jun 29 13:02 .duplicacy
-rw-r--r--  1 test10350 test10350  281 Jun 29 13:02 .user.ini
drwxr-xr-x  2 test10350 test10350 4096 Jun 29 13:02 dns
lrwxrwxrwx  1 root      root        46 Jun 29 13:03 htdocs -> /var/www/test.site/releases/release-1656507770
drwxr-xr-x  2 test10350 test10350 4096 Jun 29 13:03 logs
drwxr-xr-x  3 test10350 test10350 4096 Jun 29 13:02 modsec
drwxr-xr-x  2 test10350 test10350 4096 Jun 29 13:02 nginx
drwxr-xr-x  4 test10350 test10350 4096 Jun 29 13:02 public
drwxr-xr-x  3 test10350 test10350 4096 Jun 29 13:02 releases
-rw-r--r--  1 test10350 test10350  120 Jun 29 13:02 user-configs.php
-rw-r--r--  1 test10350 test10350 4189 Jun 29 13:02 wp-config.php
```

A `/var/www/{site.domain}/releases` diretory is created, and within that a release `/var/www/{site.domain}/release-{EPOCH}` subdirectory.

The site's files and directories are moved into the release directory, and then `htdocs` is symlinked to the release directory.

The site `wp-config.php` file is symlinked into the `releases` directory so that is available.
```
root@10alpha:/var/www/test.site# ls -la releases
total 20
drwxr-xr-x  5 test10350 test10350 4096 Jun 29 13:16 .
drwxr-xr-x+ 9 test10350 test10350 4096 Jun 29 13:16 ..
drwxr-xr-x  5 test10350 test10350 4096 Jun 29 13:03 release-1656507770
lrwxrwxrwx  1 root      root        32 Jun 29 13:16 wp-config.php -> /var/www/test.site/wp-config.php
```

You may have noticed the `.user.ini` file is now located in `/var/www/{site.domain}/.user.ini`, that is because it is now symlinked into the current release:
```
root@10alpha:/var/www/test.site# ls -la htdocs/.user.ini
lrwxrwxrwx 1 root root 28 Jun 29 13:03 htdocs/.user.ini -> /var/www/test.site/.user.ini
```
or
```
root@10alpha:/var/www/test.site# ls -la /var/www/test.site/releases/release-1656507770/.user.ini
lrwxrwxrwx 1 root root 28 Jun 29 13:03 /var/www/test.site/releases/release-1656507770/.user.ini -> /var/www/test.site/.user.ini
```
(If you are using OLS stack, this is also the case for the `.htaccess` file)

You will have also noticed the `/var/www/{site.domain}/public` directory. This directory is now for all public files, ie your uploads directory is symlinked here.
Files in this directory are not version controlled and they are not immutable.
```
root@10alpha:/var/www/test.site# ls -la htdocs/wp-content/uploads
lrwxrwxrwx 1 root root 25 Jun 29 13:03 htdocs/wp-content/uploads -> /var/www/test.site/public
```

At this point:
- The site's directory structure has been adapted for Git deployments, but the all files in all directories are the original files, your Git repo has not yet been deployed.
- The structure is also not immutable, all the files are owned by the system user, and file modifications are still allowed.

```
root@10alpha:/var/www/test.site# ls -la /var/www/test.site/releases/release-1656507770
total 220
drwxr-xr-x  5 test10350 test10350  4096 Jun 29 13:03 .
drwxr-xr-x  7 test10350 test10350  4096 Jun 29 13:28 ..
lrwxrwxrwx  1 root      root         28 Jun 29 13:03 .user.ini -> /var/www/test.site/.user.ini
-rw-r--r--  1 test10350 test10350   405 Jun 29 13:02 index.php
-rw-r--r--  1 test10350 test10350 19915 Jun 29 13:02 license.txt
-rw-r--r--  1 test10350 test10350  7401 Jun 29 13:02 readme.html
-rw-r--r--  1 test10350 test10350  7165 Jun 29 13:02 wp-activate.php
drwxr-xr-x  9 test10350 test10350  4096 Jun 29 13:02 wp-admin
-rw-r--r--  1 test10350 test10350   351 Jun 29 13:02 wp-blog-header.php
-rw-r--r--  1 test10350 test10350  2338 Jun 29 13:02 wp-comments-post.php
-rw-r--r--  1 test10350 test10350  3001 Jun 29 13:02 wp-config-sample.php
drwxr-xr-x  6 test10350 test10350  4096 Jun 29 13:03 wp-content
-rw-r--r--  1 test10350 test10350  3943 Jun 29 13:02 wp-cron.php
drwxr-xr-x 26 test10350 test10350 12288 Jun 29 13:02 wp-includes
-rw-r--r--  1 test10350 test10350  2494 Jun 29 13:02 wp-links-opml.php
-rw-r--r--  1 test10350 test10350  3973 Jun 29 13:02 wp-load.php
-rw-r--r--  1 test10350 test10350 48498 Jun 29 13:02 wp-login.php
-rw-r--r--  1 test10350 test10350  8577 Jun 29 13:02 wp-mail.php
-rw-r--r--  1 test10350 test10350 23706 Jun 29 13:02 wp-settings.php
-rw-r--r--  1 test10350 test10350 32051 Jun 29 13:02 wp-signup.php
-rw-r--r--  1 test10350 test10350  4748 Jun 29 13:02 wp-trackback.php
-rw-r--r--  1 test10350 test10350  3236 Jun 29 13:02 xmlrpc.php
```

### Directory Structure when Git Repo is deployed

Once you click Deploy, or if you have push to deploy enabled and push an update, a deploy will be triggered on your server.

After the deploy finishes, your git repo will have been pulled into a new directory, scripts will have run and if all passed, then the directory symlinks will have been updated.
However the basic directory structure will be as above.
```
root@10alpha:/var/www/test.site# ls -la
drwxr-xr-x+ 9 test10350 test10350 4096 Jun 29 13:31 .
drwxr-xr-x+ 6 root      root      4096 Jun 29 13:02 ..
drwxr--r--  3 root      root      4096 Jun 29 13:02 .duplicacy
-rw-r--r--  1 test10350 test10350  281 Jun 29 13:02 .user.ini
drwxr-xr-x  2 test10350 test10350 4096 Jun 29 13:02 dns
lrwxrwxrwx  1 root      root        46 Jun 29 13:31 htdocs -> /var/www/test.site/releases/release-1656509495
drwxr-xr-x  2 test10350 test10350 4096 Jun 29 13:31 logs
drwxr-xr-x  3 test10350 test10350 4096 Jun 29 13:02 modsec
drwxr-xr-x  2 test10350 test10350 4096 Jun 29 13:02 nginx
drwxr-xr-x  4 test10350 test10350 4096 Jun 29 13:02 public
drwxr-xr-x  8 test10350 test10350 4096 Jun 29 13:31 releases
-rw-r--r--  1 test10350 test10350  120 Jun 29 13:02 user-configs.php
-rw-r--r--  1 test10350 test10350 4321 Jun 29 13:31 wp-config.php
```

But now the site WordPress files and directories will be root owned:
```
root@10alpha:/var/www/test.site# ls -la /var/www/test.site/releases/release-1656509495
total 232
drwxr-xr-x  7 root      root       4096 Jun 29 13:31 .
drwxr-xr-x  8 test10350 test10350  4096 Jun 29 13:31 ..
drwxr-xr-x  8 root      root       4096 Jun 29 13:31 .git
drwxr-xr-x  3 root      root       4096 Jun 29 13:31 .github
-rw-r--r--  1 root      root         30 Jun 29 13:31 README.md
-rw-r--r--  1 root      root        405 Jun 29 13:31 index.php
-rw-r--r--  1 root      root      19915 Jun 29 13:31 license.txt
-rw-r--r--  1 root      root       7401 Jun 29 13:31 readme.html
-rw-r--r--  1 root      root       7165 Jun 29 13:31 wp-activate.php
drwxr-xr-x  9 root      root       4096 Jun 29 13:31 wp-admin
-rw-r--r--  1 root      root        351 Jun 29 13:31 wp-blog-header.php
-rw-r--r--  1 root      root       2338 Jun 29 13:31 wp-comments-post.php
-rw-r--r--  1 root      root       3001 Jun 29 13:31 wp-config-sample.php
drwxr-xr-x  4 root      root       4096 Jun 29 13:31 wp-content
-rw-r--r--  1 root      root       3943 Jun 29 13:31 wp-cron.php
drwxr-xr-x 26 root      root      12288 Jun 29 13:31 wp-includes
-rw-r--r--  1 root      root       2494 Jun 29 13:31 wp-links-opml.php
-rw-r--r--  1 root      root       3973 Jun 29 13:31 wp-load.php
-rw-r--r--  1 root      root      48498 Jun 29 13:31 wp-login.php
-rw-r--r--  1 root      root       8577 Jun 29 13:31 wp-mail.php
-rw-r--r--  1 root      root      23706 Jun 29 13:31 wp-settings.php
-rw-r--r--  1 root      root      32051 Jun 29 13:31 wp-signup.php
-rw-r--r--  1 root      root       4748 Jun 29 13:31 wp-trackback.php
-rw-r--r--  1 root      root       3236 Jun 29 13:31 xmlrpc.php
```
```
root@10alpha:/var/www/test.site/htdocs/wp-content# ls -la
total 20
drwxr-xr-x  4 root root 4096 Jun 29 13:31 .
drwxr-xr-x  7 root root 4096 Jun 29 13:31 ..
-rw-r--r--  1 root root   28 Jun 29 13:31 index.php
drwxr-xr-x  2 root root 4096 Jun 29 13:31 plugins
drwxr-xr-x 14 root root 4096 Jun 29 13:31 themes
lrwxrwxrwx  1 root root   25 Jun 29 13:31 uploads -> /var/www/test.site/public
```
Apart from the contents of your uploads directory, which will still be owned by the site system user:
```
root@10alpha:/var/www/test.site/htdocs/wp-content/uploads# ls -la
total 16
drwxr-xr-x  4 test10350 test10350 4096 Jun 29 13:02 .
drwxr-xr-x+ 9 test10350 test10350 4096 Jun 29 13:31 ..
drwxr-xr-x  3 test10350 test10350 4096 Jun 29 13:02 2022
```

Your site `/var/www/{site.domain}/wp-config.php` will also have been updated to include:
```
/* GridPane Git Full Immutable Start */
define('DISALLOW_FILE_MODS', true);
/* GridPane Git Full Immutable End */
```
To further restrict any file modifications from within the app.

If you disable the git integration for your site, then the directory and file structure will be returned to GridPane normal using the latest files from the current release.

# Notes

Directory structures - intermediary state
- On enable we currently check the repo and then configure a sites directories for git deployments, but do not carry out an immediate deployment.
- First deployment occurs on the push of the deploy button or trigger of the push to deploy webhook.
- This means after enable and before deploy there is an intermediary directory state where the structure is that of a git deployed site, but the files are not from the git repo.
- We understand this is superfluous and intend to combine this into a single action, however at beta stage it allows us to test and check each stage separately.

Backups
- restores should be limited to db only
- the restore archives will still contain a copy of the files if needed.

Cloning
- source site is git full deployed
  - cloning should work as normal

Clone Over
- source site is git full deployed
  - clone over should work as normal
- destination site is git full deployed
  - clone over should only allow db clones

Staging
- source site is git full deployed
  - staging should work fine, the destination site should be functional but without any traces of git
- destination site is git full deployed
  - staging should be limited to only db pushes 
