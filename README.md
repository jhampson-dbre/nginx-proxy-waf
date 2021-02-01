## Feature Proposal: Add ModSecurity web application firewall to NGINX Home Assistant SSL Proxy addon

This is a fork of the [NGINX Home Assistant SSL Proxy](https://github.com/home-assistant/addons/tree/master/nginx_proxy) add-on that includes ModSecurity web application firewall using the OWASP Core Rule Set.

**Disclaimer:** I am actively working on implementing this feature and this is currently a work in progress.

[ModSecurity](https://github.com/SpiderLabs/ModSecurity) is an open source web application firewall (WAF). When used with the [OWASP ModSecurity Core Rule Set (CRS)](https://github.com/coreruleset/coreruleset) - an open source firewall policy for ModSecurity, web application are protected from a wide range of attacks, including SQL Injection (SQLi), Cross Site Scripting (XSS), Local File Inclusion (LFI), Remote File Inclusion (RFI), and Code/Shell Injection.

From my understanding of the recent Home Assistant security disclosures, the types of vulnerabilities that were found in 3rd party custom integrations are exactly the types of attacks that ModSecurity is designed to provide a layer of protection against.

Including ModSecurity in the NGINX SSL Proxy add-on provides an easy to adopt layer of security for Home Assistant users with internet-facing installations, while only requiring minimal user configuration.

To use ModSecurity in NGINX SSL Proxy, one new configuration option is implemented:

- `security_mode`: Controls the behavior of ModSecurity web application firewall. Valid values are:
  1. `DetectionOnly` - (default) process security rules and log detections but never executes any disruptive actions (block, deny, drop)
  2. `On` - process security rules; blocks potentially malicious requests
  3. `Off` - do not process security rules

The ability easily run in "report only" mode or completely disable ModSecurity greatly reduces the user impact in case of any false positive detections or other unforeseen issues occur.

## Additional supportability considerations

1. ModSecurity is compiled from source as an nginx "dynamic module", making the initial installation a fairly complex process (compared to a simple package manager installation).
   - ModSecurity must be compiled with the source code for the version of nginx that is installed (v1.16.1 in the current NGINX SSL Proxy addon)
   - Currently the NGINX SSL Proxy addon does not specify a particular nginx version to install. If/when the version of nginx changes, ModSecurity library would need to be compiled against that version.
2. I observed that the compile process for GitHub Actions is extremely slow for ARM-based images. The amd64 build completes in ~15 minutes, but aarch64 takes around 1 hours and 45 minutes. Searching google indicates that this is a common problem.

My current solution is to use a 2-stage build:
 - Stage 1 - Build an intermediate container from Dockerfile.modsecurity that contains the compiled modsecurity libraries. ~~Although currently configured to pull from the official `nginx` repository on Docker Hub, the Home Assistant base images could be used here as well.~~ Edit: Updated `modsecurity/Dockerfile` to use Home Assistant base images.
       Stage 1 would only need to be ran:
       - to update modsecurity to new releases
       - to support newer nginx version that are introduced in Stage 2
 - Stage 2 - Build the NGINX SSL Proxy addon and copy the compiled libraries from the Stage 1 build.   Currently, the original Dockerfile now points to my (~~yet-to-published~~ Edit: modsecurity library build images are published now) Stage 1 build, but were this to be included in the core NGINX addon, it would be pointed to the images built by the Home Assistant project.
       - Stage 2 would pull the prebuilt Stage 1 copy as part of the build process to elimate the need to compile the binaries for every build.

I'm open for any other solutions to resolving the slow compiles on ARM builds.

3. The default configuration of OWASP CRS "should face [false positives] rarely, and therefore it is recommended for all sites and applications", however, if any false positives are encountered, they would need to be addressed by creating an exclusion policy and/or filing an issue in the CRS repository.

Looking forward to any feedback. If you enounter a false positive with ModSecurity blocking a legitimate action, please open a GitHub issue and include the output from addon log (with any IP addresses or other potentially sensitive information obfuscated).

Ref: [Compiling and Installing ModSecurity for NGINX Open Source](https://www.nginx.com/blog/compiling-and-installing-modsecurity-for-open-source-nginx/)