# OWASP Dependency Track - OpenShift

This template will install [OWASP Dependency Track](https://dependencytrack.org/) on OpenShift.

## Project Setup

To use Dependency Track, your app should create a [BOM](https://cyclonedx.org/) to upload to dependency track for processing.  To create a BOM in a supported format, add the following plugin to your Maven project:
```
<plugins>
    <plugin>
        <groupId>org.cyclonedx</groupId>
        <artifactId>cyclonedx-maven-plugin</artifactId>
        <version>1.3.1</version>
    </plugin>
</plugins>
```

If using Jenks supplied by OpenShift, you can install the Dependency Track [Jenkins plugin](https://plugins.jenkins.io/dependency-track) by adding the following ENVIRONMENT variable to the master Jenkins deployment config:
```
INSTALL_PLUGINS=dependency-track:2.1.0
```

When Jenkins re-deploys, it will install the Dependency Track plugin.  Once Jenkins starts, open Jenkins in a new browser tab and navigate to `Manage Jenkins -> Configure System`.

There should now be a `Dependency Track` section.  
* Add in the service URL for the Dependency Track server (e.g. http://dependency-track:8080)
* Add a token (find this by logging into Dependency Track, then navigating to `Administration -> Access Management -> Teams`)
    * Click on `Administrators`, then generate a new token, copy it, and paste it into the token input in the Jenkins config.
* Test your connection.  If it works, save and you're done.

Invoke the plugin to build the `bom.xml` file:
```
mvn org.cyclonedx:cyclonedx-maven-plugin:makeBom
```

Invoke the [Jenkins plugin](https://plugins.jenkins.io/dependency-track) to upload and process the bom file that should be found at `target/bom.xml`.

Pipeline code:
```
sh "${mvnCmd} org.cyclonedx:cyclonedx-maven-plugin:makeBom"
dependencyTrackPublisher(artifact: 'target/bom.xml', artifactType: 'bom', projectId: '<project uuid>', synchronous: false)
```
