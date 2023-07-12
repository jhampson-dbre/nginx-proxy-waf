# Add ModSecurity web application firewall to NGINX Home Assistant SSL Proxy addon

This is a fork of the [NGINX Home Assistant SSL Proxy](https://github.com/home-assistant/addons/tree/master/nginx_proxy) add-on that includes ModSecurity web application firewall using the OWASP Core Rule Set.

[ModSecurity](https://github.com/SpiderLabs/ModSecurity) is an open source web application firewall (WAF). When used with the [OWASP ModSecurity Core Rule Set (CRS)](https://github.com/coreruleset/coreruleset) - an open source firewall policy for ModSecurity, web application are protected from a wide range of attacks, including SQL Injection (SQLi), Cross Site Scripting (XSS), Local File Inclusion (LFI), Remote File Inclusion (RFI), and Code/Shell Injection.

## Why does Home Assistant need a Web Application Firewall (WAF)?

Analysis of recent Home Assistant security disclosures have shown that vulnerabilities found 3rd party custom integrations could bypass Home Assistant's authentication and allow an attacker remote access to internet exposed Home Assistant instances. These types of exploits are exactly what ModSecurity is designed to provide a layer of protection against.

Including ModSecurity in the NGINX SSL Proxy add-on provides an easy to adopt layer of security for Home Assistant users with internet-facing installations, while only requiring minimal user configuration.

## How to use Web Application Firewall with Home Assistant

To use ModSecurity in NGINX SSL Proxy, one new configuration option is implemented:

- `security.mode`: Controls the behavior of ModSecurity web application firewall. Valid values are:
  1. `DetectionOnly` - (default) process security rules and log detections but never executes any disruptive actions (block, deny, drop)
  2. `On` - process security rules; blocks potentially malicious requests

The ability easily run in "report only" mode or completely disable ModSecurity greatly reduces the user impact in case of any false positive detections or other unforeseen issues occur.

## Additional supportability considerations

1. ModSecurity is compiled from source as an nginx "dynamic module", making the initial installation a fairly complex process (compared to a simple package manager installation).
   - ModSecurity must be compiled with the source code for the version of nginx that is installed (v1.16.1 in the current NGINX SSL Proxy addon)
   - Currently the NGINX SSL Proxy addon does not specify a particular nginx version to install. If/when the version of nginx changes, ModSecurity library would need to be compiled against that version.

2. I observed that the compile process for GitHub Actions is extremely slow for ARM-based images. The amd64 build completes in ~15 minutes, but aarch64 takes around 1 hours and 45 minutes. Searching google indicates that this is a common problem.

My current solution is to use a 2-stage build:

- Stage 1 - Build an intermediate container from Dockerfile.modsecurity that contains the compiled modsecurity libraries.
       Stage 1 would only need to be ran:
       - to update modsecurity to new releases
       - to support newer nginx version that are introduced in Stage 2
- Stage 2 will pull the prebuilt Stage 1 image as part of the build process to elimate the need to compile the binaries for every build.

I'm open for any other solutions to resolving the slow compiles on ARM builds.

3. The default configuration of OWASP CRS "should face [false positives] rarely, and therefore it is recommended for all sites and applications", however, if any false positives are encountered, they would need to be addressed by creating an exclusion policy and/or filing an issue in the CRS repository.

If you enounter a false positive with ModSecurity blocking a legitimate action, please ensure the `security.debug` option is set to true and open a GitHub issue, including the debug output from addon log (with any IP addresses or other potentially sensitive information obfuscated).

Ref: [Compiling and Installing ModSecurity for NGINX Open Source](https://www.nginx.com/blog/compiling-and-installing-modsecurity-for-open-source-nginx/)
