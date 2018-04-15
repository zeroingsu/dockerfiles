## Apache Tomcat 8.0.36

A simple docker build for installing a vanilla Tomcat 8.0.36 below */opt/tomcat*. It comes out of the box and is intended for use for integration testing.

During startup a directory specified by the environment variable `DEPLOY_DIR` (*/deployments* by default) is checked for .war files. If there are any, they are linked into the *webapps/* directory for automatic deployment. 

- ### Jmx Monitor

  In order to enable Jolokia for your application you should use this image as a base image (via `FROM`) and use the output of `jmx-opts` in your startup scripts to include it in for the Java startup options.

  For example, the following snippet can be added to a script starting up your Java application

  ```
  # ...
  export JAVA_OPTIONS="$JAVA_OPTIONS $(jmx-opts)"
  # .... use JAVA_OPTIONS when starting your app, e.g. as Tomcat does

  ```

  The following versions and defaults are used:

  - [Jolokia](http://www.jolokia.org/) : version **1.5.0** and port **8778**
  - [jmx_exporter](https://github.com/prometheus/jmx_exporter): version **0.3.0** and port **9779**

You can influence the behaviour of `agent-bond-opts` by setting various environment variables:

- ### Jmx Monitor Options

  Agent bond itself can be influenced with the following environment variables:

  - **MNT_OFF** : If set disables activation of agent-bond (i.e. echos an empty value). By default,  is enabled.
  - **MNT_ENABLED** : Comma separated list of sub-agents enabled. Currently allowed values are `jolokia` and `jmx_exporter`. By default both are enabled.

  #### Jolokia configuration

  - **MNT_JOLOKIA_CONFIG** : If set uses this file (including path) as Jolokia JVM agent properties (as described in Jolokia's [reference manual](http://www.jolokia.org/reference/html/agents.html#agents-jvm)). By default this is `/opt/jolokia/jolokia.properties`.
  - **MNT_JOLOKIA_HOST** : Host address to bind to (Default: `0.0.0.0`)
  - **MNT_JOLOKIA_PORT** : Port to use (Default: `8778`)
  - **MNT_JOLOKIA_USER** : User for authentication. By default authentication is switched off.
  - **MNT_JOLOKIA_HTTPS** : Switch on secure communication with https. By default self signed server certificates are generated if no `serverCert` configuration is given in `MNT_JOLOKIA_OPTS`
  - **MNT_JOLOKIA_PASSWORD** : Password for authentication. By default authentication is switched off.
  - **MNT_JOLOKIA_ID** : Agent ID to use (`$HOSTNAME` by default, which is the container id)
  - **MNT_JOLOKIA_OPTS** : Additional options to be appended to the agent opts. They should be given in the format "key=value,key=value,..."

  Some options for integration in various environments:

  - **MNT_JOLOKIA_AUTH_OPENSHIFT** : Switch on client authentication for OpenShift TLS communication. The value of this parameter can be a relative distinguished name which must be contained in a presented client certificate. Enabling this parameter will automatically switch Jolokia into https communication mode. The default CA cert is set to`/var/run/secrets/kubernetes.io/serviceaccount/ca.crt`

  #### jmx_exporter configuration

  - **MNT_JMX_EXPORTER_OPTS** : Configuration to use for `jmx_exporter` (in the format `<port>:<path to config>`)
  - **MNT_JMX_EXPORTER_PORT** : Port to use for the JMX Exporter. Default: `9779`
  - **MNT_JMX_EXPORTER_CONFIG** : Path to configuration to use for `jmx_exporter`: Default: `/opt/agent-bond/jmx_exporter_config.json`

Features:

- Tomcat Version: **8.0.36**
- Java Base Image: **alpine-oraclejre:8u161-b12**
- Port: **8080**
- User **admin** (Password: **admin**) has been added to access the admin applications */host-manager* and */manager*)
- Documentation and examples have been removed
- Command: `/opt/tomcat/bin/deploy-and-run.sh` which links .war files from */maven* to */opt/tomcat/webapps* and then calls `undefined run`
- Sets `-Djava.security.egd=file:/dev/./urandom` for faster startup times (though a bit less secure)

### Debugging

You can enable remote debugging by setting `JAVA_DEBUG` to any value:

- **JAVA_DEBUG** If set remote debugging will be switched on
- **JAVA_DEBUG_PORT** Port used for remote debugging. Default: 5005