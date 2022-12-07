# Codefresh GitOps Certification examples - Level 2 - GitOps at scale

This repository contains examples for the ArgoCD/GitOps
certification workshops (Level 2)

Take the certification yourself at https://codefresh.io/courses/get-gitops-certified/
+++++
The App of Apps pattern
In GitOps Fundamentals, we covered how deploy a single application but what about deploying multiple related applications at once? There are two main ways to handle multiple applications, App of Apps, and Application Sets. In this module we will answer the following questions:

What problems does the App of Apps pattern solve?
When is it best to use App of Apps?
How do we use the App of Apps pattern?
How does App of Apps enable GitOps?

Argo CD allows you to define an Application resource, which is responsible for enabling the deployment and synchronization of application resources to a Kubernetes cluster.

An application defines the git repo and folder where manifests are stored, which are all of your definitions that allow your app to run in Kubernetes. Argo CD supports raw YAML manifests, custom config management and the config management tools: Kustomize, Helm, and Jsonnet.

What if we need to deploy more than one application? How do we handle those manifests? We need to create an application definition for each application being deployed, but since these apps are a group of related applications, wouldn't it be nice if Argo CD knew that?

Grouping similar applications
The Argo community came up with a solution to our questions above with the App of Apps pattern. Essentially, this pattern allows us to define a root Argo CD application that will itself define and sync multiple child applications.

Instead of pointing to an application manifest, like we were before - the Root App points to a folder in Git, where we store the application manifests that define each application we want to create and deploy. This way you are able to declare all your applications inside a single YAML manifest

Here’s an example of the pattern to give you a better understanding.


This diagram above shows how our "parent" or "root app" contains the instructions for how to deploy our child applications for My Apps. The App of Apps pattern supports Kubernetes manifests, Kustomize, and Helm. However, for simplicity's sake, we will demonstrate using plain YAML.

From a directory structure, your Git repository might look something like this:

My Apps

├── my-projects

│   ├── my-project.yml

├── my-apps

│   ├── child-app-1.yml

│   ├── child-app-2.yml

│   ├── child-app-3.yml

│   ├── child-app-4.yml

├── root-app

└───├── parent-app.yml

The root app definition would look something like this:
apiVersion: argoproj.io/v1alpha1

kind: Application

metadata:

  name: root-app

  namespace: argocd

spec:

  destination:

    namespace: my-app

    server: https://kubernetes.default.svc

  project: my-project

  source:

    path: my-apps

    repoURL: https://github.com/hseligson1/app-of-apps

    targetRevision: HEAD

The my-apps directory contains the application for each child app. Within that manifest, your file might look something like this:
apiVersion: argoproj.io/v1alpha1

kind: Application

metadata:

  name: child-app-1

  namespace: argocd

spec:

  destination:

    namespace: my-app

    server: https://kubernetes.default.svc

  project: my-project

  source:

    path: guestbook

    repoURL: https://github.com/argoproj/argocd-example-apps.git

    targetRevision: HEAD

  syncPolicy:

    syncOptions:

    - CreateNamespace=true

    automated:

      selfHeal: true

      prune: true

Within the definition above, the “path” field instructs Argo CD to look at the guestbook folder within the repo for the Kubernetes manifest. This folder contains the manifests that define our applications using any of the supported ways (Helm, Kustomize, plain YAMLs etc).

So, essentially you can use the App of Apps pattern to manage your Application resources by adding or removing Application resources via your manifests to your Git repository. This encourages a GitOps workflow and eliminates the need to operate your Argo CD applications via the Web UI or the CLI.

Use Cases of App of Apps
The main advantage of using App of Apps is that you can now treat several applications as a single unit while still keeping them isolated during deployment. The most common scenarios are

Cluster bootstrapping
Handling Argo CD applications without using the CLI or the GUI

For cluster bootstrapping, imagine that you have a set of applications that you always want to install in a new Kubernetes cluster. Instead of installing those applications one-by-one you can easily group them in a single application and then install that instead.

For the second scenario you can edit an existing root app with just git operations (e.g. add folders in the paths it looks) and have Argo CD automatically deploy applications without going through the Argo CD CLI or the Web UI.
