[![release](https://github.com/krateoplatformops/keptn-provider/actions/workflows/release.yaml/badge.svg)](https://github.com/krateoplatformops/keptn-provider/actions/workflows/release.yaml)

# Keptn operator

Ansible-base operator designed to streamline the management of Keptn projects, making it easier for DevOps teams to create, update, and maintain their projects within the Keptn framework. This operator is built using the Operator SDK, a framework that simplifies the process of creating Kubernetes-native applications, combined with Ansible, a popular automation platform.

**Key Features:**

1. **Project Lifecycle Management:** The operator empowers users to seamlessly create and manage Keptn projects through the Kubernetes API. Using custom resources, operators can effortlessly define the project's metadata, services, and integrations, allowing for consistency and repeatability in project creation.

2. **Declarative Configuration:** With Ansible integration, users can provide declarative configuration in the form of YAML files. These files outline the desired state of Keptn projects, services, and integrations, which the operator then automatically translates into actionable tasks and API calls.

3. **Automated Updates:** The operator monitors the state of Keptn projects and their components. When changes are detected in the declared configuration, the operator employs Ansible playbooks to orchestrate the necessary updates. This ensures that projects stay aligned with the desired configuration and prevents configuration drift.

4. **Intelligent Error Handling:** In the event of failures or errors during project management operations, the operator leverages Ansible's robust error handling capabilities to provide detailed diagnostics and, whenever possible, autonomously recover from the failure state.

5. **Scalability:** As a Kubernetes-native application, the operator inherits Kubernetes' scalability and resilience features. It can seamlessly manage numerous Keptn projects and adapt to fluctuations in workload.

6. **Customization:** Users can extend the operator's functionality by leveraging Ansible's vast library of modules and plugins. This flexibility allows teams to integrate the operator with their existing tools and processes.

7. **Version Control:** Keptn project configurations are often stored in version control systems like Git. The operator can pull configuration updates from these repositories, enabling teams to manage project configurations in a centralized manner.

**Use Cases:**

1. **Continuous Deployment:** DevOps teams can use the operator to automate the deployment of applications by managing the necessary Keptn projects, services, and integrations.

2. **Multi-Environment Management:** The operator can manage Keptn projects across different environments (e.g., development, staging, production), ensuring consistency and reducing human error.

3. **Infrastructure as Code:** By defining Keptn projects and configurations as code, infrastructure teams can manage projects alongside other infrastructure components.

4. **Collaborative Development:** Development and operations teams can collaborate more effectively by using version-controlled Keptn project configurations that are managed by the operator.

In conclusion, the Keptn Project Operator with Ansible Integration offers a comprehensive solution for managing Keptn projects within Kubernetes environments. By combining the strengths of the Operator SDK and Ansible, this operator simplifies project lifecycle management, configuration updates, and error handling, providing DevOps teams with a robust tool to streamline their Keptn operations.

## TODO list

- [x] Create/Update Ansible role
- [x] Add a finalizer to handle the delete event
- [x] Delete Ansible Role
- [x] Refer to Kubernetes secrets (like [here](https://kubernetes.io/docs/concepts/configuration/secret/#uses-for-secrets)) instead of clear-text secrets in the Project manifest
- [ ] Test using molecule
- [ ] Change package manager from Kustomize to Helm
- [ ] Version the Helm chart in a separate repository
- [ ] Add sync strategies to keep aligned Keptn project to the manifest

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
