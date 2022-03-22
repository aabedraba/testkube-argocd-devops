# Testing cloud native applications with ArgoCD and Testkube

This repository sets up ArgoCD to use Testkube to generate testing resource manifests that ArgoCD will then apply to a Kubernetes cluster. To make that possible, Testkube needs to be added as a plugin to ArgoCD. This repository covers the steps to do that. 

The repository is part of Testkube+ArgoCD integration which you can follow [here](https://kubeshop.io/blog/a-gitops-powered-kubernetes-testing-machine-with-argocd-and-testkube).

## 1. Building custom ArgoCD `repo-server` image

To make ArgoCD use Testkube as a plugin, the Testkube binary should be added to the `argo-repo-server` container. 

Use [`argocd-customization/argocd.dockerfile`](argocd-customization/argocd.dockerfile) to build the custom image, that uses `argocd/argocd` as the base image and then downloads and configures Testkube.

The image is built with Docker as shown below:

```sh
docker build -t CONTAINER_REPOSITORY argocd-customization/argocd.dockerfile
```

## 2. Patching ArgoCD's deployment

The `argocd-repo-server` deployment images need to be replaced by the one built in the previous step. 

Update the field `CONTAINER_REGISTRY` and apply the following the command: 

```sh
kubectl patch deployments.apps -n argocd argocd-repo-server --type json --patch-file argocd-customization/patch.yaml
```

## 3. Define Testkube in ArgoCD  PluginConfigurattionManagement

In order to define the name of the plugin that will be used by an ArgoCD application, and to define the command ArgoCD should run to generate the manifests when the plugin is used, run: 

```sh
kubectl patch configmaps -n argocd argocd-cm --patch-file argocd-customization/argocd-plugins.yaml
```

## 4. Create an ArgoCD application that uses the Testkube plugin 

In [`testkube-application.yaml`](testkube-application.yaml) update the fieldd:
 -  `REPOSITORY_URL` with the Git repository containing your test definitions 
 - `TESTS_PATH_IN_REPOSITORY` with the path to the tests folder

Create the application by running the command

```sh
kubectl apply -f testkube-application.yaml
```