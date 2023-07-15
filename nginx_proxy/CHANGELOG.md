# Changelog

## 5.0.4

Fix startup debug config on/off inversion

## 5.0.3

Make enabling startup debugging configurable

## 5.0.2

Add debugging for startup error

## 5.0.1

Fix s6-overlay error due to breaking change in latest base image

## 5.0.0

BREAKING CHANGE: Nginx is updated to version 1.24.0

Other noteworth changes:

- Bump Alpine version to 3.18
- Build ModSecurity from [2cb6344](https://github.com/SpiderLabs/ModSecurity/tree/2cb6344bf8a1f7d564d45d6d6a3745212abdd37f)
- Build Core Rule Set from [ec00051](https://github.com/coreruleset/coreruleset/tree/ec0005183329f0241a31c8bd136598023d9aa8aa)

## 4.1.1

Fix incorrect path to modsecurity rules inclusion file

## 4.1.0

Add new `security.customize` option to allow for creating custom ModSecurity exclusion rules. See the docs for details.

## 4.0.1

Fix add-on image name format

## 4.0.0

BREAKING CHANGE: Nginx is updated to version 1.20.2. As part of this update, we are switching from the default Alpine nginx package to the nginx.org package to streamline the process to install ModSecurity. The nginx.org package is only available for aarch64 and amd64. Therefore, the following CPU architectures are no longer supported:

- armhf
- armv7
- i386

Other noteworth changes:

- Bump Alpine version to 3.14
- Pull add-on base image from ghcr instead of docker hub

## 3.0.20

- Add additional security exclusion rule for false positive when saving files with File Editor addon

## 3.0.19

- Make security.debug optional to fix add-on startup failure after upgrade
## 3.0.18

- Add additional security exclusion rule for false positive when using Grafana/InfluxDB addons

## 3.0.17

- Add security exclusion rule for false positive when using Grafana/InfluxDB addons
- Add security.debug option to display detailed info about requests blocked by ModSecurity

## 3.0.16

- Add security exclusion rule for false positive when starting File Editor addon

## 3.0.15

- Move data files to rootfs

## 3.0.14

- Bump nginx-116-modsecurity to 0.0.3

## 3.0.12

- Fix syntax for run.sh

## 3.0.11

- Refactor configuration options for ModSecurity to improve flexibility

## 3.0.1

- Fix the use of subfolders with certificate files

## 3.0

- Update Alpine to 3.11
- Use mozilla Recommended SSL settings

## 2.6

- Remove ipv6 listener because we run only inside virtual network on a ipv4 range

## 2.5

- Migrate to Bashio

## 2.4

- Added Cloudflare mechanism for creating auto-generated ipv4/ipv6 list for real visitor ip

## 2.3

- Fix issue with nginx warning for ssl directive

## 2.2

- Fix issue with `homeassistant` connection
- Update nginx to version 1.16.1

## 2.1

- Update nginx to version 1.14.2

## 2.0

- Update nginx to version 1.14.0

## 1.2

- Modify `server_names_hash_bucket_size` to add support for longer domain names

## 1.1

- Update run.sh info messages
- Make HSTS configurable and optional

## 1.0

- Add customization mechanism using included config snippets from /share
- Optimize logo.png
- Update base image
