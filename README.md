# Cloud Foundry Droplet Building Buildpack
## Purpose of buildpack
The purpose of this buildpack is to enable users to download from Cloud Foundry the complete application, including application binaries, and deploy exactly the same application to a different environment. This is particularly useful in a CI/CD pipeline where the operator wants to guarantee that exactly the same build of an application gets deployed to all environments. In that way a change or update to the buildpack will not create a difference between what gets deployed in different environments.

## Instructions to run test applications
This repository includes both the buildpack and two samples that leverages the application. The samples can be found in the [/testapps](/testapps) directory. These samples include the customization required to use this buildpack, as well as helper scripts to make it easier to run the samples.

To run the test applications, do the following steps:
* Navigate to /testapps directory

  ```
  $ cd testapps/
  ```
* Build and push application using regular buildpack, wrapped by the droplet-builder buildpack.
  
  ```
  $ ./push-spring-music.sh
  ...
  
  $ ./push-static-hellow-world.sh
  ```
  You now two demo applications built and deployed, leveraging the [Static Buildpack](https://github.com/cloudfoundry/staticfile-buildpack) and the [Java Buildpack](https://github.com/cloudfoundry/java-buildpack)
* Download droplets from Cloud Foundry

  ```
  $ ./download-static-hello-world.sh
  ...
  $ ./download-spring-music.sh
  ```
  You'll now have the droplets downloaded in a format that makes it easy to deploy the same droplet using the [Binary Buildpack](https://github.com/cloudfoundry/binary-buildpack)
* Push the Droplet versions of the applications back into Cloud Foundry

  ```
  $ cd static-hello-world-droplet
  $ cf p
  $ cd ..
  $ cd spring-music-droplet
  $ cf p
  ```
  
