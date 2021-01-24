## Feature Proposal: Add ModSecurity web application firewall to NGINX Home Assistant SSL Proxy addon

**Disclaimer:** I am actively working on implementing this feature and this pull request is currently a work in progress. Since this is a fairly significant work effort, I wanted to get some early feedback from the project maintainers on if this is something that would be considered for inclusion in the core NGINX Home Assistant SSL proxy addon so I can ensure any questions or criticisms are addressed.

ModSecurity is an open source web application firewall (WAF). When used with the OWASP ModSecurity Core Rule Set (CRS) - essentially the firewall policy for ModSecurity, web application are protected from a wide range of attacks, including SQL Injection (SQLi), Cross Site Scripting (XSS), Local File Inclusion (LFI), Remote File Inclusion (RFI), and Code/Shell Injection.

From my understanding of the recent Home Assistant security disclosures, the types of vulnerabilities that were found in 3rd party custom integrations are exactly the types of attacks that ModSecurity is designed to provide a layer of protection against.

Including ModSecurity in the NGINX SSL Proxy add-on would provide an easy to adopt layer of security for Home Assistant users with internet accessible installations, while only requiring minimal user configuration.

To implement ModSecurity in NGINX SSL Proxy, some potential additional configurations that would be useful are:
1. Enable/disable the WAF (set in nginx.conf) -
- `modsecurity off` - modsecurity is disabled. detection and/or prevention is not performed
- `modsecurity on` - modsecurity is enabled. detection and/or prevention will be performed per the run mode (see below)
2. Set the WAF run mode (set in modsec/modsecurity.conf)
- `SecRuleEngine On` - actively drops requests detected as malicious
- `SecRuleEngine DetectionOnly` - malicious requests are detected and logged, but not dropped

The ability easily run in "report only" mode or completely disable ModSecurity greatly reduces the user impact in case of any false positive detections or other unforeseen issues occur.

## Additional supportability considerations

1. ModSecurity is compiled from source as an nginx "dynamic module", making the initial installation a fairly complex process (compared to a simple package manager installation).
   - ModSecurity must be compiled with the source code for the version of nginx that is installed (v1.16.1 in the current NGINX SSL Proxy addon)
   - Currently the NGINX SSL Proxy addon does not specify a particular nginx version to install. If/when the version of nginx changes, ModSecurity library would need to be compiled against that version.
2. I observed that the compile process for GitHub Actions is extremely slow for ARM-based images. The amd64 build completes in ~15 minutes, but aarch64 takes around 1 hours and 45 minutes. Searching google indicates that this is a common problem.

My current solution is to use a 2-stage build:
    1. Stage 1 - Build an intermediate container from Dockerfile.modsecurity that contains the compiled modsecurity libraries. Although currently configured to pull from the official `nginx` repository on Docker Hub, the Home Assistant base images could be used here as well.
       Stage 1 would only need to be ran
       - to update modsecurity to new releases
       - to support newer nginx version that are introduced in Stage 2
    2. Stage 2 - Build the NGINX SSL Proxy addon and copy the compiled libraries from the Stage 1 build.   Currently, the original Dockerfile now points to my (yet-to-published) Stage 1 build, but were this to be included in the core NGINX addon, it would be pointed to the images built by the Home Assistant project.
       Stage 2 would pull the prebuilt Stage 1 copy as part of the build process to elimate the need to compile the binaries for every build.

I'm open for any other solutions to this problem.

3. The default configuration of OWASP CRS "should face [false positives] rarely, and therefore it is recommended for all sites and applications", however, if any false positives are encountered, they would need to be addressed by creating an exclusion policy and/or filing an issue in the CRS repository.

## Alternatives considered for securing Home Assistant with ModSecurity

A community fork of the NGINX SSL Proxy addon could be created that includes ModSecurity. While a fork would provide the community with an additional option, ModSecurity should be considered for inclusion in the core NGINX Home Assistant SSL proxy addon because:
1. Anyone running the core NGINX Home Assistant SSL proxy addon to expose there Home Assistant to the Internet could benefit from the additional protection provided by ModSecurity WAF.
2. Anyone who does not want the protection provided by ModSecurity or encounters any problems due to false positives can disable the protection and continue to use the existing functionality.
3. While recently disclosed security vulnerabilities have been mitigated, ModSecurity would provide one more security layer to guard against yet-to-be-discovered vulnerabilities.


Thanks for taking the time to review this proposal. Looking forward to any feedback.