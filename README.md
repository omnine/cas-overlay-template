CAS Overlay Template [![Build Status](https://travis-ci.org/apereo/cas-overlay-template.svg?branch=master)](https://travis-ci.org/apereo/cas-overlay-template)
=======================

Generic CAS WAR overlay to exercise the latest versions of CAS. This overlay could be freely used as a starting template for local CAS war overlays.

# Versions

- CAS `6.3.x`
- JDK `11`

# Overview

To build the project, use:

```bash
# Use --refresh-dependencies to force-update SNAPSHOT versions
./gradlew[.bat] clean build
```

To see what commands are available to the build script, run:

```bash
./gradlew[.bat] tasks
```

To launch into the CAS command-line shell:

```bash
./gradlew[.bat] downloadShell runShell
```

To fetch and overlay a CAS resource or view, use:

```bash
./gradlew[.bat] getResource -PresourceName=[resource-name]
```

To list all available CAS views and templates:

```bash
./gradlew[.bat] listTemplateViews
```

To unzip and explode the CAS web application file and the internal resources jar:

```bash
./gradlew[.bat] explodeWar
```

# Configuration

- The `etc` directory contains the configuration files and directories that need to be copied to `/etc/cas/config`.

```bash
./gradlew[.bat] copyCasConfiguration
```

- The specifics of the build are controlled using the `gradle.properties` file.

## Adding Modules

CAS modules may be specified under the `dependencies` block of the [Gradle build script](build.gradle):

```gradle
dependencies {
    implmentation "org.apereo.cas:cas-server-some-module:${project.casVersion}"
    ...
}
```

To collect the list of all project modules and dependencies:

```bash
./gradlew[.bat] allDependencies
```

You could also add modules and dependencies dynamically on the fly using the `casModules` project property. For example, to include support for OpenID Connect and Duo Security, you could invoke the build using `-PcasModules=oidc,duo` and have it auto-include modules that provide requested functionality. Needless, to say, you will need to know the module name beforehand.

### Clear Gradle Cache

If you need to, on Linux/Unix systems, you can delete all the existing artifacts (artifacts and metadata) Gradle has downloaded using:

```bash
# Only do this when absolutely necessary
rm -rf $HOME/.gradle/caches/
```

Same strategy applies to Windows too, provided you switch `$HOME` to its equivalent in the above command.

# Deployment

- Create a keystore file `thekeystore` under `/etc/cas`. Use the password `changeit` for both the keystore and the key/certificate entries. This can either be done using the JDK's `keytool` utility or via the following command:

```bash
./gradlew[.bat] createKeystore
```

- Ensure the keystore is loaded up with keys and certificates of the server.

On a successful deployment via the following methods, CAS will be available at:

* `https://cas.server.name:8443/cas`

## Executable WAR

Run the CAS web application as an executable WAR:

```bash
./gradlew[.bat] run
```

Debug the CAS web application as an executable WAR:

```bash
./gradlew[.bat] debug
```

Run the CAS web application as a *standalone* executable WAR:

```bash
./gradlew[.bat] clean executable
```

## External

Deploy the binary web application file `cas.war` after a successful build to a servlet container of choice.

## Docker

The following strategies outline how to build and deploy CAS Docker images.

### Jib

The overlay embraces the [Jib Gradle Plugin](https://github.com/GoogleContainerTools/jib) to provide easy-to-use out-of-the-box tooling for building CAS docker images. Jib is an open-source Java containerizer from Google that lets Java developers build containers using the tools they know. It is a container image builder that handles all the steps of packaging your application into a container image. It does not require you to write a Dockerfile or have Docker installed, and it is directly integrated into the overlay.

```bash
./gradlew build jibDockerBuild
```

### Dockerfile

You can also use the native Docker tooling and the provided `Dockerfile` to build and run CAS.

```bash
chmod +x *.sh
./docker-build.sh
./docker-run.sh
```

# DualShield Authentication Delegation

**CAS** can be configured to delegate authentication to an external identity provider, e.g. **DualShield**. We tested it(6.3) under Ubuntu 18.04.

### JDK 11
Install it by `sudo apt-get install openjdk-11-jdk`

### Tomcat 9
See the following links.  
[How To Install Apache Tomcat 9 on Ubuntu 18.04](https://www.digitalocean.com/community/tutorials/install-tomcat-9-ubuntu-1804)  
[How to Install Tomcat 9 on Ubuntu 18.04](https://linuxize.com/post/how-to-install-tomcat-9-on-ubuntu-18-04/)  
For TLS, modify `server.xml` accordingly.
```aidl
<Connector port="8443" protocol="HTTP/1.1" SSLEnabled="true" maxThreads="150" scheme="https" secure="true"
    keystoreFile="the-path-to/certificate.pfx" keystorePass="password"
    clientAuth="false" keystoreType="PKCS12" sslEnabledProtocols="TLSv1,TLSv1.1,TLSv1.2"/>
```

### CAS Overlay
Customize it by adding the following line in `build.gradle`  

    `implementation "org.apereo.cas:cas-server-support-pac4j-webflow:${casServerVersion}"`  

Then build it.  

    `./gradlew clean build`

The output `cas.war` (under the subfolder `build/libs` ) can be deployed to a servlet container like Apache Tomcat.  
By the way, you can also run it in place with the embedded container via the following command:

`java -jar /path/to/cas.war`

### Configuration
Create a folder `/etc/cas/config`, copy the sample file `cas.properties` under the subfolder `etc/cas/config/` into it.
You can modify the content.  
Get DualShield IDP metadata, name it as `idp-metadata.xml`, then save it into `/etc/cas/config/`.

### Deploy cas.war
Copy it to Tomcat webapps folder, e.g. `/opt/tomcat/latest/webapps`, restart Tomcat service.   
The SP metadata `sp-metadata.xml` will be generated under the folder `/etc/cas/config`
```aidl
root@nano190018:/etc/cas/config# ls -ltr
total 32
-rw-r--r-- 1 tomcat tomcat 2071 Mar 18 17:33 idp-metadata.xml
-rw-r----- 1 tomcat tomcat 2475 Mar 19 11:45 samlKeystore.jks
-rw-r----- 1 tomcat tomcat 1019 Mar 19 11:45 saml-signing-cert-DualShield.pem
-rw-r----- 1 tomcat tomcat 1746 Mar 19 11:45 saml-signing-cert-DualShield.key
-rw-r----- 1 tomcat tomcat  679 Mar 19 11:45 saml-signing-cert-DualShield.crt
-rw-r----- 1 tomcat tomcat 6088 Mar 19 11:45 sp-metadata.xml
-rw-rw-r-- 1 tomcat tomcat 1001 Mar 19 13:17 cas.properties
```
You can also obtain it from `https://cas.deepnetsecurity.com:8443/cas/sp/metadata`, which you should register it in DualShield.

### Test it
Access it in browser by, `https://cas.deepnetsecurity.com:8443/cas`

### References

[How CAS Works](https://calnetweb.berkeley.edu/calnet-technologists/cas/how-cas-works)  
[CAS web flow](https://apereo.github.io/cas/development/images/cas_flow_diagram.png)  
[SafeNet Trusted Access for Apereo CAS](https://resources.safenetid.com/help/Apereo%20CAS/Apereo_CAS_Help/Index.htm)  
[Apereo CAS - Delegated Authentication to SAML2 Identity Providers](https://apereo.github.io/2019/02/25/cas61-delegate-authn-saml2-idp/)  
[CAS 5 SAML2 Delegated AuthN Tutorial](https://apereo.github.io/2017/03/22/cas51-delauthn-tutorial/)