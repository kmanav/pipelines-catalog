# Buildah (PipelineResource)

This Task builds source into a container image using Project Atomic's
[Buildah](https://github.com/projectatomic/buildah) build tool. It uses
Buildah's support for building from
[`Dockerfile`](https://docs.docker.com/engine/reference/builder/)s, using its
`buildah bud` command. This command executes the directives in the `Dockerfile`
to assemble a container image, then pushes that image to a container registry.

## Install the Task

```
kubectl apply -f https://raw.githubusercontent.com/openshift/pipelines-catalog/master/task/buildah-pr/0.1/buildah-pr.yaml
```

## Inputs

### Parameters

* **BUILDER_IMAGE:**: The name of the image containing the Buildah tool. See
  note below.  (_default:_ quay.io/buildah/stable:v1.17.0)
* **DOCKERFILE**: The path to the `Dockerfile` to execute (_default:_
  `./Dockerfile`)
* **CONTEXT**: Path to the directory to use as context (_default:_
  `.`)
* **TLSVERIFY**: Verify the TLS on the registry endpoint (for push/pull to a
  non-TLS registry) (_default:_ `true`)
* **FORMAT**: The format of the built container, oci or docker (_default:_
 `oci`)


### Resources

* **source**: A `git`-type `PipelineResource` specifying the location of the
  source to build.

## Outputs

### Resources

* **image**: An `image`-type `PipelineResource` specify the image that should
  be built.

## Creating a ServiceAccount

Buildah builds an image and pushes it to the destination registry which is
defined as a parameter. The image needs proper credentials to be
authenticated by the remote container registry. These credentials can
be provided through a serviceaccount. See [Authentication](https://github.com/tektoncd/pipeline/blob/master/docs/auth.md#basic-authentication-docker)
for further details.

If you run on OpenShift, you also need to allow the service
account to run privileged containers. Due to security considerations
OpenShift does not allow containers to run as privileged containers
by default.

Run the following in order to create a service account named
`pipelines` on OpenShift and allow it to run privileged containers:

```
oc create serviceaccount pipeline
oc adm policy add-scc-to-user privileged -z pipeline
oc adm policy add-role-to-user edit -z pipeline
```

## Creating the taskrun

This TaskRun runs the Task to fetch a Git repo, and build and push a container
image using Buildah.

```
apiVersion: tekton.dev/v1beta1
kind: TaskRun
metadata:
  name: buildah-build-my-repo
spec:
  # Use service account with git and image repo credentials
  serviceAccountName: pipeline
  taskRef:
    name: buildah-pr
  resources:
    inputs:
    - name: source
      resourceSpec:
        type: git
        params:
        - name: url
          value: https://github.com/my-user/my-repo
    outputs:
    - name: image
      resourceSpec:
        type: image
        params:
        - name: url
          value: gcr.io/my-repo/my-image
```

In this example, the Git repo being built is expected to have a `Dockerfile` at
the root of the repository.