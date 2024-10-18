# Example Publish Central Gradle

This is example repository publish Java application to [Central Portal](https://central.sonatype.com/) 
using Gradle. This example use Gradle plugin [gradle-maven-publish-plugin](https://vanniktech.github.io/gradle-maven-publish-plugin/central/). 
Pre-requisite for this would be to have a namespace created and verified in Central Portal. 


## GPG Key Generation

GPG key is required to sign the artifacts. This step creates the gpg key. Extract the key

```bash
# Generate key
gpg --gen-key

# List key and verify key has been created and note the key id
gpg --list-keys

# Distribute public key. Following servers can be used to distribute the keys
# keyserver.ubuntu.com keys.openpgp.org pgp.mit.edu
gpg --keyserver keyserver.ubuntu.com --send-keys <email/key id>

# Verify key has been sent to the keyservers with following command
gpg --keyserver keyserver.ubuntu.com --recv-keys <email/key id>

# If the ring has multiple keys it is neccessary to specify the secret key when signing
# So list the secret keys and make a note of the key id
gpg --list-secret-keys

# Export the private key armor to configure as environment variable
# Value will be set in environment variable ORG_GRADLE_PROJECT_signingInMemoryKey
gpg --export-secret-keys --armor <email/key id> | grep -v '\-\-' | grep -v '^=.' | tr -d '\n'

# Find the private key id. This will be the short string in sec line after the key type rsa
# sec   rsa4096/5DD49C32 2020-06-24 [SC] [expired: 2024-06-24]
# Key id is 5DD49C32
gpg --list-secret-keys --keyid-format short <email/key id>
```

## Gradle Configuration

This is the extract of the [build.gradle](./app/build.gradle) file which are relevant for the Central 
Portal publish.

```java
import com.vanniktech.maven.publish.JavaLibrary
import com.vanniktech.maven.publish.JavadocJar
import com.vanniktech.maven.publish.SonatypeHost

plugins {
    id 'application'
    id 'maven-publish'
    id "com.vanniktech.maven.publish" version "0.30.0"
}

repositories {
    // Use Maven Central for resolving dependencies.
    mavenCentral()
}

mavenPublishing {
  coordinates("name.divitotawela.sandbox", "app", "0.0.1")

  pom {
    name = "App"
    description = "Example app"
    inceptionYear = "2024"
    url = "https://github.com/cdivitotawela/example.publish-central-gradle"
    licenses {
      license {
        name = "The Apache License, Version 2.0"
        url = "http://www.apache.org/licenses/LICENSE-2.0.txt"
        distribution = "http://www.apache.org/licenses/LICENSE-2.0.txt"
      }
    }
    developers {
      developer {
        id = "username"
        name = "Chaminda"
        url = "https://github.com/cdivitotawela/"
      }
    }
    scm {
      url = "https://github.com/cdivitotawela/example.publish-central-gradle"
      connection = "scm:git:git://github.com/cdivitotawela/example.publish-central-gradle.git"
      developerConnection = "scm:git:ssh://git@github.com/cdivitotawela/example.publish-central-gradle.git"
    }
  }

  publishToMavenCentral(SonatypeHost.CENTRAL_PORTAL)

  signAllPublications()
}
```

## Publish Artifacts

Use the Gradle command to publish the artifacts

```bash
## Export environment variables
# These can be get from the Central Portal
ORG_GRADLE_PROJECT_mavenCentralUsername=username
ORG_GRADLE_PROJECT_mavenCentralPassword=the_password

# These we extracted from above steps
ORG_GRADLE_PROJECT_signingInMemoryKey=exported_ascii_armored_key
ORG_GRADLE_PROJECT_signingInMemoryKeyId=12345678

# This is the passphrase used for the GPG key
ORG_GRADLE_PROJECT_signingInMemoryKeyPassword=some_password

## Publish
./gradlew publishToMavenCentral
```

## References
- https://vanniktech.github.io/gradle-maven-publish-plugin/central/
- https://central.sonatype.org/publish/publish-portal-gradle/