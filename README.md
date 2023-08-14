[![release](https://github.com/krateoplatformops/keptn-provider/actions/workflows/release.yaml/badge.svg)](https://github.com/krateoplatformops/keptn-provider/actions/workflows/release.yaml)

# Keptn operator

## TODO list

- [x] Create/Update Ansible role
- [x] Add a finalizer to handle the delete event
- [x] Delete Ansible Role
- [x] Refer to Kubernetes secrets (like [here](https://kubernetes.io/docs/concepts/configuration/secret/#uses-for-secrets)) instead of clear-text secrets in the Project manifest
- [ ] Test using molecule
- [ ] Change package manager from Kustomize to Helm
- [ ] Version the Helm chart in a separate repository

## Steps to create this operator

1. Initialize the operator and the API dedicated to the Keptn project

    ```bash
    operator-sdk init --plugins=ansible --domain=krateo.io

    operator-sdk create api --group keptn --version v1alpha1 --kind Project --generate-role
    ```

2. Add the Ansible role `projectfin` reference to `watches.yaml` for the finalizers to handle the delete event of the CRD

    ```yaml
   - version: v1alpha1
     group: keptn.krateo.io
     kind: Project
     role: project
     finalizer:
       name: finalizer.projects.keptn.krate.io
       role: projectfin
    ```

3. Fix the reference to the docker image (and tag) in `Makefile` 

    ```bash
    VERSION ?= 0.0.1
    ...
    IMAGE_TAG_BASE ?= ghcr.io/krateoplatformops/keptn-provider
    ...
    IMG ?= $(IMAGE_TAG_BASE):$(VERSION)

    ```

4. Implement two different roles `project` and `projectfin`

    > **Note**: The names of all variables in the spec field are converted to snake_case by the operator before running Ansible. For example, serviceAccount in the spec becomes service_account in Ansible. You can disable this case conversion by setting the snakeCaseParameters option to false in your watches.yaml. It is recommended that you perform some type of validation in Ansible on the variables to ensure that your application is receiving the expected input.

5. Adding and pushing a new tag triggers the CI pipeline, producing a new docker image of the controller manager
6. (Only for Mac ARM users) Add Makefile target to `Makefile` for build and push docker image for manual testing purposes

   ```make
   BUILDPLATFORM ?= linux/amd64
   .PHONY: docker-buildx-push
   docker-buildx-push: ## Build and push docker image for the manager for cross-platform support
       docker buildx build --push --platform=${BUILDPLATFORM} --tag ${IMG} -f Dockerfile .
   ```

   and execute the command to build and push the image to the registry:

   ```bash
   make docker-buildx-push
   ```

7. Once the docker image will be pushed to the registry, execute this command to deploy the operator to the Kubernetes context in use

    ```bash
    make deploy
    ```
