![policy_as_code_war](./assets/policy_as_code_war.png)

# Introduction
In this guide, we are going to demonstrate what OPA Gatekeeper and Kyverno are, what are the differences between them and how we can set up and use them in the Kubernetes cluster by doing hands-on demo. 

So, if you are interested in with one of these topics, please keep reading, there is a lots of good details in the following sections 💪.

Let's start with defining what Policy-as-Code concept is.

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

- 🧰 [Prerequisites](#prerequisites)
- 🛡️ [What is Policy-as-Code?](#what-is-policy-as-code)
- <img src="https://github.com/open-policy-agent/opa/blob/master/logo/logo.svg" height="16" width="16"/> [What is OPA Gatekeeper ?](#what-is-opa-gatekeeper-)
- <img src="https://github.com/kyverno/artwork/blob/main/Kyverno.svg" height="16" width="16"/> [What is Kyverno ?](#what-is-kyverno-)
- 🎭 [What are differences between OPA Gatekeeper and Kyverno ?](#what-are-differences-between-opa-gatekeeper-and-kyverno-)
- 🧑‍💻 [Hands On](#hands-on)
- 👀 [References](#references)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# Prerequisites

* <img src="./assets/minikube.svg" height="16" width="16"/> minikube v1.17.1
* <img src="https://github.com/cncf/artwork/blob/master/other/illustrations/ashley-mcnamara/kubectl/kubectl.svg" height="16" width="16"/> kubectl v1.20.2

# What is Policy-as-Code?
Similar to the concept of `Infrastructure-as-Code (IaC)` and the benefits you get from codifying your infrastructure setup using the software development practices, `Policy-as-Code (PaC)` is the codification of your policies. 

PaC is the idea of writing code in a high-level language to manage and automate policies. By representing policies as code in text files, proven software development best practices can be adopted such as version control, automated testing, and automated deployment. 

The policies you want to enforce come from your organization’s established guidelines or agreed-upon conventions, and best practices within the industry. It could also be derived from tribal knowledge that has accumulated over the years within your operations and development teams.

PaC is very general, so, it can be applied to any environment that you want to manage and enforce policies but if want to apply it unto the Kubernetes world, there are two tools became very important: OPA Gatekeeper and Kyverno.

Let's continue with the description of these tools.

# What is OPA Gatekeeper ?
Before move on with the description of the OPA Gatekeeper, we should explain the OPA (Open Policy Agent) is first.

The [OPA](https://github.com/open-policy-agent/opa) is an open-source, general-purpose policy engine that can be used to enforce policies on various types of software systems like microservices, CI/CD pipelines, gateways, Kubernetes, etc. OPA was developed by Styra and is currently a part of CNCF.

The [OPA Gatekeeper](https://github.com/open-policy-agent/gatekeeper) is the policy controller for Kubernetes. More technically, it is a customizable Kubernetes Admission Webhook that helps enforce policies and strengthen governance.

The important thing that we should notice is the use of OPA is not tied to the Kubernetes alone. OPA Gatekeeper, on the other hand, is specifically built for Kubernetes Admission Control use case of OPA.

# What is Kyverno ?
[Kyverno](https://github.com/kyverno/kyverno/) is a policy engine designed for Kubernetes. With Kyverno, policies are managed as Kubernetes resources and no new language is required to write policies. This allows using familiar tools such as kubectl, git, and kustomize to manage policies. Kyverno policies can validate, mutate, and generate Kubernetes resources. The Kyverno CLI can be used to test policies and validate resources as part of a CI/CD pipeline. Kyverno is an open-source and a part of CNCF Sandbox Project also.

# What are differences between OPA Gatekeeper and Kyverno ?
Let's explain these differences with the table format.

| Features/Capabilities                       	| Gatekeeper 	| Kyverno 	|
|---------------------------------------------	|------------	|---------	|
| Validation                                  	|      ✓     	|    ✓    	|
| Mutation                                    	|     ✓*     	|    ✓    	|
| Generation                                  	|      X     	|    ✓    	|
| Policy as native resources                  	|      ✓     	|    ✓    	|
| Metrics exposed                             	|      ✓     	|    ✓    	|
| OpenAPI validation schema (kubectl explain) 	|      X     	|    ✓    	|
| High Availability                           	|      ✓     	|    ✓    	|
| API object lookup                           	|      ✓     	|    ✓*   	|
| CLI with test ability                       	|     ✓**    	|    ✓    	|
| Policy audit ability                        	|      ✓     	|    ✓    	|

`* Alpha status`
`** Separate CLI`

> Credit: https://neonmirrors.net/post/2021-02/kubernetes-policy-comparison-opa-gatekeeper-vs-kyverno/

In my opinion, the best advantages of using Kyverno are no need to learn another policy language and the OpenAPI validation schema support that we can use via kubectl explain command. On the other hand side, OPA Gatekeeper has lots of tools developed around the Rego language to help us to write and test our policies such as [conftest](https://github.com/instrumenta/conftest), [konstraint](https://github.com/plexsystems/konstraint) and this is a big plus in my opinion. These are the tools that we can use to implement `Policy-as-Code Pipeline`. Another advantage of using OPA Gatekeeper, therese are lots of libraries that includes ready to use policies written for us such as [gatekeeper-library](https://github.com/open-policy-agent/gatekeeper-library), [konstraint-examples](https://github.com/plexsystems/konstraint/tree/main/examples) and [raspbernetes-policies](https://github.com/raspbernetes/k8s-security-policies/tree/master/policies).

# Hands On
I created two seperate folders for OPA Gatekeeper and Kyverno resources. We are going to start with the OPA Gatekepeer project first.

There are various types of installation of OPA Gatekeeper but in this section we are going to use [plain YAML manifest](./opa-gatekeeper/deploy.yaml) to install it. Let's install OPA Gatekeeper using the YAML manifest. In order to do that, we need to start our local Kubernetes cluster using `minikube`, we are going to use two different [Minikube profiles](https://minikube.sigs.k8s.io/docs/commands/profile/) for the OPA Gatekeeper and the Kyverno, that will result with the creating two seperate Kubernetes cluster.
```bash
$ minikube start -p opa-gatekeeper
😄  [opa-gatekeeper] minikube v1.17.1 on Darwin 10.15.7
✨  Using the hyperkit driver based on user configuration
👍  Starting control plane node opa-gatekeeper in cluster opa-gatekeeper
🔥  Creating hyperkit VM (CPUs=3, Memory=8192MB, Disk=20000MB) ...
🌐  Found network options:
    ▪ no_proxy=127.0.0.1,localhost
🐳  Preparing Kubernetes v1.20.2 on Docker 20.10.2 ...
    ▪ env NO_PROXY=127.0.0.1,localhost
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔎  Verifying Kubernetes components...
🌟  Enabled addons: storage-provisioner, default-storageclass
🏄  Done! kubectl is now configured to use "opa-gatekeeper" cluster and "default" namespace by default
```

Let's apply the manifest.
```bash
$ kubectl apply -f opa-gatekeeper/deploy.yaml
namespace/gatekeeper-system created
Warning: apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
customresourcedefinition.apiextensions.k8s.io/configs.config.gatekeeper.sh created
customresourcedefinition.apiextensions.k8s.io/constraintpodstatuses.status.gatekeeper.sh created
customresourcedefinition.apiextensions.k8s.io/constrainttemplatepodstatuses.status.gatekeeper.sh created
customresourcedefinition.apiextensions.k8s.io/constrainttemplates.templates.gatekeeper.sh created
serviceaccount/gatekeeper-admin created
podsecuritypolicy.policy/gatekeeper-admin created
role.rbac.authorization.k8s.io/gatekeeper-manager-role created
clusterrole.rbac.authorization.k8s.io/gatekeeper-manager-role created
rolebinding.rbac.authorization.k8s.io/gatekeeper-manager-rolebinding created
clusterrolebinding.rbac.authorization.k8s.io/gatekeeper-manager-rolebinding created
secret/gatekeeper-webhook-server-cert created
service/gatekeeper-webhook-service created
deployment.apps/gatekeeper-audit created
deployment.apps/gatekeeper-controller-manager created
Warning: admissionregistration.k8s.io/v1beta1 ValidatingWebhookConfiguration is deprecated in v1.16+, unavailable in v1.22+; use admissionregistration.k8s.io/v1 ValidatingWebhookConfiguration
validatingwebhookconfiguration.admissionregistration.k8s.io/gatekeeper-validating-webhook-configuration created
```

You should notice that bunch of CRDs created to allow define and enforce policies called `ConstraintTemplate` which describes both the Rego that enforces the constraint and the schema of the constraint.

In this section, we are going to enforce policy to validate required labels that we want to on resources, if required label exits then we'll approve the request, if not we'll reject it.

Let's look at the `ConstraintTemplate` that we are going to apply.
```yaml
apiVersion: templates.gatekeeper.sh/v1beta1
kind: ConstraintTemplate
metadata:
  name: k8srequiredlabels
spec:
  crd:
    spec:
      names:
        kind: K8sRequiredLabels
      validation:
        # Schema for the `parameters` field
        openAPIV3Schema:
          properties:
            labels:
              type: array
              items: string
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8srequiredlabels

        violation[{"msg": msg, "details": {"missing_labels": missing}}] {
          provided := {label | input.review.object.metadata.labels[label]}
          required := {label | label := input.parameters.labels[_]}
          missing := required - provided
          count(missing) > 0
          msg := sprintf("you must provide labels: %v", [missing])
        }
```

You should notice that the policy that we define with the Rego language is placed under the `.targets[].rego` section. Once we applied this to the cluster, `K8sRequiredLabels` Custom Resource is going to be created and by using this CR we'll define our policy context, means which resources we want to apply the policy on.

Let's apply it.
```bash
$ kubectl apply -f opa-gatekeeper/k8srequiredlabels-constraint-template.yaml
constrainttemplate.templates.gatekeeper.sh/k8srequiredlabels created

$ kubectl get customresourcedefinitions.apiextensions.k8s.io
Found existing alias for "kubectl". You should use: "k"
NAME                                                 CREATED AT
configs.config.gatekeeper.sh                         2021-02-25T09:06:10Z
constraintpodstatuses.status.gatekeeper.sh           2021-02-25T09:06:10Z
constrainttemplatepodstatuses.status.gatekeeper.sh   2021-02-25T09:06:10Z
constrainttemplates.templates.gatekeeper.sh          2021-02-25T09:06:10Z
k8srequiredlabels.constraints.gatekeeper.sh          2021-02-25T09:19:39Z
```

As you can see, `K8sRequiredLabels` CR is created. Lets define and apply it too.
```yaml
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequiredLabels
metadata:
  name: ns-must-have-gk
spec:
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Namespace"]
  parameters:
    labels: ["gatekeeper"]
```

You should notice that we'll enforce the policy on `Namespace` resource and the label value that we want to be available on the Namespace is `gatekepeer`.
```bash
$ kubectl apply -f opa-gatekeeper/k8srequiredlabels-constraint.yaml
k8srequiredlabels.constraints.gatekeeper.sh/ns-must-have-gk created
```

Let's test with creating invalid namespace then a valid one.
```bash
$ kubectl apply -f opa-gatekeeper/invalid-namespace.yaml
Found existing alias for "kubectl apply -f". You should use: "kaf"
Error from server ([denied by ns-must-have-gk] you must provide labels: {"gatekeeper"}): error when creating "opa-gatekeeper/invalid-namespace.yaml": admission webhook "validation.gatekeeper.sh" denied the request: [denied by ns-must-have-gk] you must provide labels: {"gatekeeper"}
```

```bash
$ kubectl apply -f opa-gatekeeper/valid-namespace.yaml
Found existing alias for "kubectl apply -f". You should use: "kaf"
namespace/valid-namespace created
```

Tadaaaa, it worked 🎉🎉🎉🎉

Let's move on with the Kyverno, again, there are various way to install it unto the Kubernetes, in this case, we are going to use Helm. We said that we'll start up another Minikub cluster with different profile.
Let's start with it.
```bash
$ minikube start -p kyverno
😄  [kyverno] minikube v1.17.1 on Darwin 10.15.7
✨  Using the hyperkit driver based on user configuration
👍  Starting control plane node kyverno in cluster kyverno
🔥  Creating hyperkit VM (CPUs=3, Memory=8192MB, Disk=20000MB) ...
🌐  Found network options:
    ▪ no_proxy=127.0.0.1,localhost
🐳  Preparing Kubernetes v1.20.2 on Docker 20.10.2 ...
    ▪ env NO_PROXY=127.0.0.1,localhost
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔎  Verifying Kubernetes components...
🌟  Enabled addons: storage-provisioner, default-storageclass
🏄  Done! kubectl is now configured to use "kyverno" cluster and "default" namespace by default

$ minikube profile list
|----------------|-----------|---------|---------------|------|---------|---------|-------|
|    Profile     | VM Driver | Runtime |      IP       | Port | Version | Status  | Nodes |
|----------------|-----------|---------|---------------|------|---------|---------|-------|
| kyverno        | hyperkit  | docker  | 192.168.64.17 | 8443 | v1.20.2 | Running |     1 |
| minikube       | hyperkit  | docker  | 192.168.64.15 | 8443 | v1.20.2 | Stopped |     1 |
| opa-gatekeeper | hyperkit  | docker  | 192.168.64.16 | 8443 | v1.20.2 | Running |     1 |
|----------------|-----------|---------|---------------|------|---------|---------|-------|
```

Let's install it by using Helm.
```bash
$ helm repo add kyverno https://kyverno.github.io/kyverno/
"kyverno" has been added to your repositories

$ helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "kyverno" chart repository
...Successfully got an update from the "nats" chart repository
...Successfully got an update from the "falcosecurity" chart repository
...Successfully got an update from the "openfaas" chart repository
...Successfully got an update from the "stable" chart repository
Update Complete. ⎈Happy Helming!⎈

$ helm install kyverno --namespace kyverno kyverno/kyverno --create-namespace
NAME: kyverno
LAST DEPLOYED: Thu Feb 25 13:16:21 2021
NAMESPACE: kyverno
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Thank you for installing kyverno 😀

Your release is named kyverno.

We have installed the "default" profile of Pod Security Standards and set them in audit mode.

Visit https://kyverno.io/policies/ to find more sample policies.
```

Let's look at the Custom Resource Definitions list.
```bash
$ kubectl get customresourcedefinitions.apiextensions.k8s.io
Found existing alias for "kubectl". You should use: "k"
NAME                                     CREATED AT
clusterpolicies.kyverno.io               2021-02-25T10:16:16Z
clusterpolicyreports.wgpolicyk8s.io      2021-02-25T10:16:16Z
clusterreportchangerequests.kyverno.io   2021-02-25T10:16:16Z
generaterequests.kyverno.io              2021-02-25T10:16:16Z
policies.kyverno.io                      2021-02-25T10:16:16Z
policyreports.wgpolicyk8s.io             2021-02-25T10:16:16Z
reportchangerequests.kyverno.io          2021-02-25T10:16:16Z
```

We can also use `kubectl explain` command to get information easily about the resource using the OpenAPI schema.
```bash
$ kubectl explain policies
KIND:     Policy
VERSION:  kyverno.io/v1

DESCRIPTION:
     Policy declares validation, mutation, and generation behaviors for matching
     resources. See: https://kyverno.io/docs/writing-policies/ for more
     information.

FIELDS:
   apiVersion	<string>
     APIVersion defines the versioned schema of this representation of an
     object. Servers should convert recognized schemas to the latest internal
     value, and may reject unrecognized values. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources

   kind	<string>
     Kind is a string value representing the REST resource this object
     represents. Servers may infer this from the endpoint the client submits
     requests to. Cannot be updated. In CamelCase. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds

   metadata	<Object>
     Standard object's metadata. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata

   spec	<Object> -required-
     Spec defines policy behaviors and contains one or rules.

   status	<Object>
     Status contains policy runtime information.
```

Lets look at our first policy definition. In this case we are using validating feature of Kyverno.
```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-labels
spec:
  validationFailureAction: enforce
  rules:
  - name: check-for-labels
    match:
      resources:
        kinds:
        - Pod
    validate:
      message: "label `app.kubernetes.io/name` is required"
      pattern:
        metadata:
          labels:
            app.kubernetes.io/name: "?*"
```
You should notice that we enforcing a required label policy on Pod resource. We are ddefining policies using native Kyverno Custom Resource called `ClusterPolicy`.

Let's apply it.
```bash
$ kubectl apply -f kyverno/validating/requirelabels-clusterpolicy.yaml
clusterpolicy.kyverno.io/require-labels created
```

Let's test it by creating a Deployment that violates the policy.
```bash
$ kubectl apply -f kyverno/validating/invalid-deployment.yaml
Found existing alias for "kubectl apply -f". You should use: "kaf"
Error from server: error when creating "kyverno/validating/invalid-deployment.yaml": admission webhook "validate.kyverno.svc" denied the request:

resource Deployment/default/nginx was blocked due to the following policies

require-labels:
  autogen-check-for-labels: 'validation error: label `app.kubernetes.io/name` is required. Rule autogen-check-for-labels failed at path /spec/template/metadata/labels/app.kubernetes.io/name/'
```

Let's apply valid one.
```bash
$ kubectl apply -f kyverno/validating/valid-deployment.yaml
pod/nginx created

$ kubectl get pods
NAME    READY   STATUS              RESTARTS   AGE
nginx   0/1     ContainerCreating   0          6s
```

Tadaaaa, it worked 🎉🎉🎉🎉

# References
* https://medium.com/trendyol-tech/enforce-organizational-policies-and-security-best-practices-to-your-kubernetes-clusters-by-using-dfc085528e07
* https://www.velotio.com/engineering-blog/deploy-opa-on-kubernetes
* https://engineering.mercari.com/en/blog/entry/20201222-enhance-kubernetes-security-with-opa-gatekeeper/
* https://docs.hashicorp.com/sentinel/concepts/policy-as-code
* https://www.magalix.com/blog/policy-as-code-for-kubernetes
* https://betterprogramming.pub/policy-as-code-on-kubernetes-with-kyverno-b144749f144
* https://itnext.io/fitness-validation-for-your-kubernetes-apps-policy-as-code-7fad698e7dec
