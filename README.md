# 3scale Configuration - GitOps Tutorial

The following tutorial provide steps on leveraging GitOps to configure 3scale using 
3scale CRs (Product, Backend CRs)

## Prerequisites
- 3scale operator being installed either should be cluster wide operator or if namespace specific then the operator should be installed in the namespace where the 3scale CRs are to be applied using GitOps
- Install 3scale using APIManager CR

## Install RH OpenShift GitOps
Install Red Hat OpenShift GitOps operator from OperatorHub from the OCP webconsole

![](images/gitops-operator.png)

This will automatically create 
- `openshift-gitops` namespace 
- `ArgoCD` CR with the name `openshift-gitops` is created in openshift-gitops namespace.
 `ArgoCD` CR creates a bunch of deployments. These deployments together make ArgoCD application
- `AppProject` CR with the name `default` is created in openshift-gitops namespace.

## Login to OpenShift GitOps
Go to the login page using the route URL it creates as `openshift-gitops-server`.

You can login to `OpenShift GitOps` using the `admin` user that comes with ArgoCD deployment OR using
OpenShift as oauth provider by clicking the button `LOG IN VIA OPENSHIFT`.

Find the password for the admin in `openshift-gitops-cluster` secret in `openshift-gitops` namespace.

## Enabling RBAC
Create cluster role to create, update, delete 3scale CRs (Need to have admin access to OCP for this)

```
oc apply -f rbac/ClusterRole_gitops-threescale-access.yaml
```
Assign the cluster role to sa `openshift-gitops-argocd-application-controller`

```
oc adm policy add-role-to-user gitops-threescale-access system:serviceaccount:openshift-gitops:openshift-gitops-argocd-application-controller -n <namespace where 3scale CRs to be applied>
```

## Create ArgoCD Application
Create the ArgoCD Application

```
oc apply -f gitops/Application_threescale-dev.yaml -n openshift-gitops
```

## Connect the git repository

Configure the repositories to be connected by the ArgoCD application 

Click `Manager your repositories, projects, settings` icon on the left panel of the ArgoCD console, Click 
`Repositories` and Click either `Connect repo using SSH` OR `Connect repo using HTTPS` and fill in the form as shown below

![](images/gitops-connectrepo.png)

## 3scale CRs
**Please note that the directory structure used for 3scale CRs are for the tutorial purpose only. Please adjust based on the needs**

3scale CRs required for this tutorial are spread under multiple directories and uses kustomize plugin 
to replace/merge the environment specific values

- `base` - 3scale manifest attributes that are common for all environments are housed here (spread under different directories i.e. `products`, `backends` etc.)

- `overlays/dev` - 3scale manifest attributes that are specific for dev environment and different from base are housed here (spread under different directories i.e. `products`, `backends` etc.)