<img align="right" src="https://raw.githubusercontent.com/startxfr/docker-images/master/travis/logo-small.svg?sanitize=true">

# docker-images-example-bash


Example of a simple bash application using the startx s2i builder [startx/fedora](https://hub.docker.com/r/startx/fedora). 
For more information on how to use this image, **[read startx OS image guideline](https://github.com/startxfr/docker-images/blob/master/OS/README.md)**.

## Running this example in OKD (aka Openshift)

### Create a sample application

```bash
# Create a openshift project
oc new-project startx-example-bash
# start a new application (build and run)
oc process -f https://raw.githubusercontent.com/startxfr/docker-images/master/Services/bash/openshift-template-build.yml -p APP_NAME=myapp | oc create -f -
# Watch when resources are available
sleep 10 && oc get all
```

### Create a personalized application

- **Initialize** a project
  ```bash
  export MYAPP=myapp
  oc new-project ${MYAPP}
  ```
- **Add template** to the project service catalog
  ```bash
  oc create -f https://raw.githubusercontent.com/startxfr/docker-images/master/Services/bash/openshift-template-build.yml -n startx-example-bash
  ```
- **Generate** your current application definition
  ```bash
  export MYVERSION=0.1
  oc process -n startx-example-bash -f startx-bash-build-template \
      -p APP_NAME=v${MYVERSION} \
      -p APP_STAGE=example \
      -p BUILDER_TAG=latest \
      -p SOURCE_GIT=https://github.com/startxfr/docker-images-example-bash.git \
      -p SOURCE_BRANCH=master \
      -p RUN_ACTION=run \
      -p MEMORY_LIMIT=256Mi \
  > ./${MYAPP}.definitions.yml
  ```
- **Review** your resources definition stored in `./${MYAPP}.definitions.yml`
- **build and run** your application
  ```bash
  oc create -f ./${MYAPP}.definitions.yml -n startx-example-bash
  sleep 15 && oc get all
  ```
- **Test** your application
  ```bash
  oc logs pods
  ```

## Running this example with source-to-image (aka s2i)

### Create a sample application

```bash
# Build the application
s2i build https://github.com/startxfr/docker-images-example-bash startx/fedora startx-bash-sample
# Run the application
docker run --rm -d startx-bash-sample
# Test the sample application
  docker ps  
```

### Create a personalized application

- **Initialize** a project directory
  ```bash
  git clone https://github.com/startxfr/docker-images-example-bash.git bash-myapp
  cd bash-myapp
  rm -rf .git
  ```
- **Develop** and create a personalized script
  ```bash
  cat << "EOF"
  echo "I'm running fo 10 sec ..."; sleep 10; exit;
  EOF > run
  chmod ugo+x run
  ```
- **Build** your current application with startx bash builder
  ```bash
  s2i build . startx/fedora:latest startx-bash-myapp
  ```
- **Run** your application and test it
  ```bash
  docker run --rm -d startx-bash-myapp 
  ```
- **Test** your application
  ```bash
  docker ps  
  ```
