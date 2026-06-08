# Magento 2.4.9 / PHP 8.5 Notes

## What changed

- `composer.json` now allows `magento/magento2-base` `>=2.4.9 <2.4.10`.
- `dist/Dockerfile` now uses a PHP 8.5 base image.

The Composer package name remains `phoenix-media/magento2-cloud-build`.

## Base image

The selected base image is:

```text
ghcr.io/sak0a/nginx-php:8.5
```

`phoenixmedia/nginx-php:8.5` was checked with `docker manifest inspect` and was not accessible, so a forked base image was used.

## docker-nginx-php fork

`PHOENIX-MEDIA/docker-nginx-php` was forked to:

```text
https://github.com/sak0a/docker-nginx-php
```

The `php85` branch changes the Dockerfile to:

- Use `php:8.5-fpm-alpine`.
- Use `composer:2.9`.
- Skip compiling `opcache`, because Zend OPcache is already provided by the PHP 8.5 base image.

The fork also includes `.github/workflows/build-php85.yml`, which builds the branch Dockerfile and publishes `ghcr.io/sak0a/nginx-php:8.5` with GitHub Actions package-write permissions.

## Composer installation

Add this repository override to the Magento project's `composer.json`:

```json
{
    "repositories": [
        {
            "type": "vcs",
            "url": "https://github.com/sak0a/magento2-cloud-build"
        }
    ],
    "require": {
        "phoenix-media/magento2-cloud-build": "dev-php85-magento249 as 1.0.50"
    }
}
```

Then run:

```bash
composer update phoenix-media/magento2-cloud-build magento/ece-tools --with-all-dependencies
vendor/bin/px-cloud-build-install
```

## Docker verification

After installing the fork in the Magento project, verify the generated Dockerfile and image:

```bash
grep '^FROM' Dockerfile
docker build --pull -t magento-php85-test .
docker run --rm magento-php85-test php -v
docker run --rm magento-php85-test composer --version
```
