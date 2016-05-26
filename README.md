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
  
## How to modify your own application to leverage this buildpack
This example will leverage an example application found at https://github.com/cloudfoundry-samples/test-app.

1. Create an emtpy directory as your work area.

   ```
   [/myworkarea]$
   ```
2. Clone the sample application from GitHub

   ```
  [/myworkarea]$ git clone https://github.com/cloudfoundry-samples/test-app
  Cloning into 'test-app'...
  remote: Counting objects: 265, done.
  remote: Total 265 (delta 0), reused 0 (delta 0), pack-reused 265
  Receiving objects: 100% (265/265), 80.75 KiB | 0 bytes/s, done.
  Resolving deltas: 100% (98/98), done.
  Checking connectivity... done.
  ```
3. Take a look at the test application
   
   ```
   [/myworkarea]$ cd test-app
   [/myworkarea/test-app]$ ls -l
   total 64
   -rw-r--r--  1 bboe  wheel    100 May 25 16:14 Dockerfile
   drwxr-xr-x  5 bboe  wheel    170 May 25 16:14 Godeps
   -rw-r--r--  1 bboe  wheel  11325 May 25 16:14 LICENSE
   -rw-r--r--  1 bboe  wheel     14 May 25 16:14 Procfile
   -rw-r--r--  1 bboe  wheel    954 May 25 16:14 README.md
   -rwxr-xr-x  1 bboe  wheel    229 May 25 16:14 build.sh
   drwxr-xr-x  9 bboe  wheel    306 May 25 16:14 handlers
   drwxr-xr-x  3 bboe  wheel    102 May 25 16:14 helpers
   -rw-r--r--  1 bboe  wheel   2372 May 25 16:14 main.go
   drwxr-xr-x  3 bboe  wheel    102 May 25 16:14 routes
   ```
   This is a web application written in Go.
4. This application is ready to be deployed as is, which can be done with the following command:

   ```
   [/myworkarea/test-app]$ cf push test-app
   ...
   ```
5. Modify the application to leverage the Droplet Buildpack.
   In order to extract the Droplet we actually need to run through the push process within Cloud Foundry with the buildpack appropriate for the application being deployed, and the extract the application binaries and additional information on how to run the application.
   This is done by using the Droplet Builder buildpack to gather this extra information.
   The test application does not come with a manifest.yml, so we need to create one, which will look as follows:
   
   ```
   ---
   applications:
   - name: test-app
      buildpack: https://github.com/bboe-pivotal/droplet-builder
      env:
        deploybuildpack: https://github.com/cloudfoundry/go-buildpack
   ```
   The above contains the bare minimum manifest.yml for this example.
   Note the following about this manifest.yml:
   * The buildpack for this application is set to https://github.com/bboe-pivotal/droplet-builder.
   * An extra environment variable refers to the buildpack that the application actually relies on to run.
     This environment variable must refer to the Git-source of the buildpack. It can not refer to a built in buildpack in Cloud Foundry.
6. Push the modified application into Cloud Foundry

   ```
   [/myworkarea/test-app]$ cf push
   ...
   requested state: started
   instances: 1/1
   usage: 256M x 1 instances
   urls: test-app.local.pcfdev.io
   last uploaded: Wed May 25 20:29:28 UTC 2016
   stack: unknown
   buildpack: https://github.com/bboe-pivotal/droplet-builder

        state     since                    cpu    memory      disk        details
   #0   running   2016-05-25 04:30:00 PM   0.0%   0 of 256M   0 of 512M
   ```
   The application have been deployed, runs as normal and can be tested at the URL specified by Cloud Foundry. In addition to that has the buildpack left behind some extra files in the deployment that can be downloading through SSH directly from one of the application instances.
7. Download droplet and generate deployable executable. This operation relies on a script called [downloaddroplet](bin/downloaddroplet). 
8. Clone the droplet-builder repsitory in the work area.

   ```
   [/myworkarea/test-app]$ cd ..
   [/myworkarea]$ git clone https://github.com/bboe-pivotal/droplet-builder/
   Cloning into 'droplet-builder'...
   remote: Counting objects: 333, done.
   remote: Total 333 (delta 1), reused 1 (delta 1), pack-reused 331
   Receiving objects: 100% (333/333), 133.95 KiB | 0 bytes/s, done.
   Resolving deltas: 100% (98/98), done.
   Checking connectivity... done.
   ```
9. Create a new directory for the droplet

   ```
   [/myworkarea]$ mkdir test-app-droplet
   [/myworkarea]$ cd test-app-droplet
   [/myworkarea/test-app-droplet]$
   ```
10. Generate droplet using script from buildpack. This script requires reference to the application that was just deployed to Cloud Foundry and the original manifest.yml used earlier. The original manifest.yml is needed to make sure the droplet can be deployed with all original settings for things like services, environment variables, memory, ec.
    
    ```
    [/myworkarea/test-app-droplet]$ ../droplet-builder/bin/downloaddroplet test-app ../test-app/manifest.yml
    Constructing manifest.yml
    Environment yaml section not set, remove declaration
    Constructing app folder
    ```
    The directory now has a new manifest.yml that has all the old settings, but have been modified to use the binary buildpack instead. The app folder contains what will actually get pushed into Cloud Foundry, which includes a tar-ball containing the Droplet and a Procfile needed by the binary buildpack with information on how to start the application.
11. Push the Droplet-based version of the application to Cloud Foundry

    ```
    [/myworkarea/test-app-droplet]$ cf push
    ...
    App test-app was started using this command `tar xzf app.tar.gz && rm app.tar.gz && test-app`

    Showing health and status for app test-app in org pcfdev-org / space dev as admin...
    OK

    requested state: started
    instances: 1/1
    usage: 256M x 1 instances
    urls: test-app.local.pcfdev.io
    last uploaded: Thu May 26 13:51:00 UTC 2016
    stack: unknown
    buildpack: binary_buildpack

         state     since                    cpu    memory      disk        details
    #0   running   2016-05-26 09:51:07 AM   0.0%   0 of 256M   0 of 512M
    ```
    Note that the application is now up and running using the binary buildpack and is now ready to be tested.
