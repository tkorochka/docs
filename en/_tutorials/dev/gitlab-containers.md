[{{ GL }}](https://about.gitlab.com/) is a tool for Continuous integration (CI).

This tutorial describes:
* Building an application into a Docker container.
* Deploying an application from a container in a [{{ managed-k8s-full-name }} cluster](../../managed-kubernetes/concepts/index.md#kubernetes-cluster) via {{ GL }} using the {{ yandex-cloud }} tools.

Each commit to {{ GL }} is followed by:
* Running a script that includes steps to build the [Docker image](../../container-registry/concepts/docker-image.md).
* Applying a new {{ managed-k8s-name }} cluster configuration specifying the application to deploy.

To set up the infrastructure required to store the source code, build the Docker image, and deploy your applications, follow these steps:
1. [Prepare your cloud](#before-you-begin).

   
   1. [Review the list of paid resources available](#paid-resources).


   1. [Install additional dependencies](#prepare).
1. [Create {{ managed-k8s-name }} and {{ container-registry-full-name }} resources](#k8s-cr-create).
1. [Create a {{ GL }} instance](#create-gitlab).
1. [Configure {{ GL }}](#configure-gitlab).
1. [Create a test application](#app-create).
1. [Create a {{ GLR }}](#runners).
1. [Create a {{ GLA }}](#agent).
1. [Configure building and deploying a Docker image from CI](#ci).

If you no longer need the resources you created, [delete them](#clear-out).

## Prepare your cloud {#before-you-begin}

{% include [before-you-begin](../_tutorials_includes/before-you-begin.md) %}


### Required paid resources {#paid-resources}

Infrastructure support costs include fees for the following resources:
* [Disks](../../compute/concepts/disk.md) and continuously running [VMs](../../compute/concepts/vm.md) (see the [{{ compute-full-name }} pricing](../../compute/pricing.md)).
* Usage of a dynamic [public IP](../../vpc/concepts/ips.md) (see [{{ vpc-full-name }} pricing](../../vpc/pricing.md)).
* Storage of created Docker images (see [{{ container-registry-name }} pricing](../../container-registry/pricing.md)).
* Usage of a [{{ managed-k8s-name }} master](../../managed-kubernetes/concepts/index.md#master) (see [{{ managed-k8s-name }} pricing](../../managed-kubernetes/pricing.md)).


### Install additional dependencies {#prepare}

To run the script, install the following in the local environment:
* [{{ yandex-cloud }} command line interface (YC CLI)](../../cli/operations/install-cli.md)
* [`jq` JSON stream processor](https://stedolan.github.io/jq/)
* [Helm package manager]({{ links.helm.install }})

## Create {{ managed-k8s-name }} and {{ container-registry-name }} resources {#k8s-cr-create}

1. Create a {{ managed-k8s-name }} cluster and a {{ container-registry-name }} registry.

   Create the {{ managed-k8s-name }} resources required to run the script: a [cluster](../../managed-kubernetes/concepts/index.md#kubernetes-cluster) and [node group](../../managed-kubernetes/concepts/index.md#node-group).

   To store Docker images, you need a {{ container-registry-name }} [registry](../../container-registry/concepts/registry.md).

   {% list tabs %}

   - Manually

      1. If you do not have a [network](../../vpc/concepts/network.md#network) yet, [create one](../../vpc/operations/network-create.md).
      1. If you do not have any [subnets](../../vpc/concepts/network.md#subnet), [create them](../../vpc/operations/subnet-create.md) in the [availability zones](../../overview/concepts/geo-scope.md) where your {{ managed-k8s-name }} cluster and node group will be created.
      1. [Create service accounts](../../iam/operations/sa/create.md):
         * With the [{{ roles-editor }}](../../iam/concepts/access-control/roles.md#editor) role to the folder where a {{ managed-k8s-name }} cluster is being created. The resources the {{ managed-k8s-name }} cluster needs will be created on behalf of this account.
         * With the [{{ roles-cr-puller }}](../../iam/concepts/access-control/roles.md#cr-images-puller) and [{{ roles-cr-pusher }}](../../iam/concepts/access-control/roles.md#cr-images-pusher) roles. This service account will be used to push the Docker images that you build to {{ GL }} and pull them to run in pods.

         {% note tip %}

         You can use the same [service account](../../iam/concepts/users/service-accounts.md) to manage your {{ managed-k8s-name }} cluster and its node groups.

         {% endnote %}

      1. [Create a {{ managed-k8s-name }} cluster](../../managed-kubernetes/operations/kubernetes-cluster/kubernetes-cluster-create.md) and a [node group](../../managed-kubernetes/operations/node-group/node-group-create.md) with the following settings:
         * **Service account for resources**: Select the service account with the `{{ roles-editor }}` role you created previously.
         * **Service account for nodes**: Select the service account with the `{{ roles-cr-puller }}` and `{{ roles-cr-pusher }}` roles that you created previously.
         * **{{ k8s }} version**: Select **1.21** or higher.
         * **Public address**: `Auto`.

         Save the cluster ID, as you will need it at the next steps.
      1. [Create a registry](../../container-registry/operations/registry/registry-create.md).

   - Using {{ TF }}

      1. If you do not have {{ TF }} yet, [install it](../../tutorials/infrastructure-management/terraform-quickstart.md#install-terraform).
      1. Download [the file with provider settings](https://github.com/yandex-cloud/examples/tree/master/tutorials/terraform/provider.tf). Place it in a separate working directory and [specify the parameter values](../../tutorials/infrastructure-management/terraform-quickstart.md#configure-provider).
      1. Download the [k8s-argocd.tf](https://github.com/yandex-cloud/examples/tree/master/tutorials/terraform/managed-kubernetes/k8s-argocd.tf) cluster configuration file to the same working directory. The file describes:
         * [Network](../../vpc/concepts/network.md#network).
         * [Subnet](../../vpc/concepts/network.md#subnet).
         * [Security group](../../vpc/concepts/security-groups.md) and [rules](../../managed-kubernetes/operations/connect/security-groups.md) needed to run the {{ managed-k8s-name }} cluster:
            * Rules for service traffic.
            * Rules for accessing the {{ k8s }} API and managing the cluster with `kubectl` through ports 443 and 6443.
            * Rules for connecting to a Git repository over SSH on port 22.
            * Rules that allow HTTP and HTTPS traffic through ports 80 and 443.
            * Rules for connecting to {{ container-registry-name }} through port 5050.
         * {{ managed-k8s-name }} cluster.
         * [Service account](../../iam/concepts/users/service-accounts.md) required to use the {{ managed-k8s-name }} cluster and node group.
         * [{{ container-registry-name }} registry](../../container-registry/concepts/registry.md).
      1. Specify the following in the configuration file:
         * [Folder ID](../../resource-manager/operations/folder/get-id.md).
         * {{ k8s }} version for the {{ managed-k8s-name }} cluster and node groups.
         * {{ managed-k8s-name }} cluster CIDR.
         * Name of the cluster service account.
         * Name of the {{ container-registry-name }} registry.
      1. Run the `terraform init` command in the directory with the configuration files. This command initializes the provider specified in the configuration files and enables you to use the provider resources and data sources.
      1. Make sure the {{ TF }} configuration files are correct using this command:

         ```bash
         terraform validate
         ```

         If there are any errors in the configuration files, {{ TF }} will point them out.
      1. Create the required infrastructure:

         {% include [terraform-apply](../../_includes/mdb/terraform/apply.md) %}

         {% include [explore-resources](../../_includes/mdb/terraform/explore-resources.md) %}

   {% endlist %}

   {% note warning %}

   For applications running in production environments, make sure to restrict access of {{ k8s }} cluster service accounts to pushing images to a registry. This is required for security reasons. In that case, create a separate service account with the `{{ roles-cr-pusher }}` role and use it for building containers.

   {% endnote %}

1. [Create a static key](../../iam/operations/sa/create-access-key.md) for the service account with the `{{ roles-cr-pusher }}` role and save it to the `key.json` file:

   ```bash
   yc iam key create \
     --service-account-name <service_account_name> \
     --output key.json
   ```

   This key is required to access the registry from {{ GL }}.

1. [Save the ID of the registry created](../../container-registry/operations/registry/registry-list.md#registry-get), as you will need it at the next steps.

1. {% include [kubectl-install-links](../../_includes/managed-kubernetes/kubectl-install.md) %}

## Create a {{ GL }} instance {#create-gitlab}

{% list tabs %}


- {{ mgl-full-name }} instance

   Create a [{{ mgl-name }} instance](../../managed-gitlab/concepts/index.md#instance) by following [this guide](../../managed-gitlab/quickstart.md#instance-create).


- VM running a {{ GL }} image

   Launch {{ GL }} on a VM with a public IP.

   {% include [create-gitlab](../../_includes/managed-gitlab/create.md) %}

{% endlist %}

## Configure {{ GL }} {#configure-gitlab}

{% include [Create a project](../../_includes/managed-gitlab/initialize.md) %}

## Create a test application {#app-create}

Create a test application that can be deployed in a {{ managed-k8s-name }} cluster:
1. Add `Dockerfile` to the project:
   1. Log in to {{ GL }}.
   1. On the home page, select a repository.
   1. Select the **Repository** → **Files** section.
   1. Click **+** and select **New file** from the drop-down menu.
   1. Name the file as `Dockerfile` and add the following code to it:

      ```Dockerfile
      FROM alpine:3.10
      CMD echo "Hello"
      ```

   1. Add a comment to the commit in the **Commit message** field: `Dockerfile for test application`.
   1. Click **Commit changes**.
1. Add the manifest for the {{ managed-k8s-name }} cluster resources to the project:
   1. Select the **Repository** → **Files** section.
   1. Click **+** and select **New file** from the drop-down menu.
   1. Name the file as `k8s.yaml`:

      ```yaml
      apiVersion: v1
      kind: Namespace
      metadata:
        name: hello-world
      ---
      apiVersion: apps/v1
      kind: Deployment
      metadata:
        name: hello-world-deployment
        namespace: hello-world
      spec:
        replicas: 1
        selector:
          matchLabels:
            app: hello
        template:
          metadata:
            namespace: hello-world
            labels:
              app: hello
          spec:
            containers:
              - name: hello-world
                image: {{ registry }}/<registry_ID>/hello:__VERSION__
                imagePullPolicy: Always
      ```

   1. In the `<registry_ID>` field, specify the ID of the registry that you created previously.
   1. Add a comment to the commit in the **Commit message** field: `Docker image deployment config`.
   1. Click **Commit changes**.

## Create a {{ GLR }} {#runners}

{% include notitle [create glr](../../_includes/managed-gitlab/k8s-runner.md) %}

## Configure a Docker image build and deployment from CI {#ci}

{% include [Setup CI/CD](../../_includes/managed-gitlab/configure-ci.md) %}

## Delete the resources you created {#clear-out}

Some resources are not free of charge. Delete the resources you no longer need to avoid paying for them:

1. [Delete the created Docker images](../../container-registry/operations/docker-image/docker-image-delete.md).
1. Delete the {{ managed-k8s-name }} cluster and {{ container-registry-name }} registry:

   {% list tabs %}

   - Manually

      1. [Delete the {{ managed-k8s-name }} cluster](../../managed-kubernetes/operations/kubernetes-cluster/kubernetes-cluster-delete.md).
      1. [Delete the {{ container-registry-name }} registry](../../container-registry/operations/registry/registry-delete.md).
      1. [Delete the created subnets](../../vpc/operations/subnet-delete.md) and [networks](../../vpc/operations/network-delete.md).
      1. [Delete the created service accounts](../../iam/operations/sa/delete.md).

   - Using {{ TF }}

      1. In the command line, go to the directory with the current {{ TF }} configuration file with an infrastructure plan.
      1. Delete the `k8s-gl.tf` configuration file.
      1. Make sure the {{ TF }} configuration files are correct using this command:

         ```bash
         terraform validate
         ```

         If there are any errors in the configuration files, {{ TF }} will point them out.
      1. Confirm updating the resources.

         {% include [terraform-apply](../../_includes/mdb/terraform/apply.md) %}

         All the resources described in the `k8s-gl.tf` configuration file will be deleted.

   {% endlist %}

1. [Delete the {{ GL }} VM](../../compute/operations/vm-control/vm-delete.md) or {{ mgl-name }} instance that you created.

## See also {#see-also}

* [Creating test VMs using {{ GL }} CI](../../tutorials/testing/ci-for-snapshots.md).