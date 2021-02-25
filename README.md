![policy_as_code_war](./assets/policy_as_code_war.png)

# Introduction
In this guide, we are going to demonstrate what OPA Gatekeeper and Kyverno are, what are the differences between them and how we can set up and use them in the Kubernetes cluster by doing hands-on demo. 

So, if you are interested in with one of these topics, please keep reading, there is a lots of good details in the following sections 💪.

Let's start with defining what Policy-as-Code concept is.

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

- 🛡️ [What is Policy-as-Code?](#what-is-policy-as-code)
- <img src="https://github.com/open-policy-agent/opa/blob/master/logo/logo.svg" height="16" width="16"/> [What is OPA Gatekeeper ?](#what-is-opa-gatekeeper-)
- <img src="https://github.com/kyverno/artwork/blob/main/Kyverno.svg" height="16" width="16"/> [What is Kyverno ?](#what-is-kyverno-)
- 🎭 [What are differences between OPA Gatekeeper and Kyverno ?](#what-are-differences-between-opa-gatekeeper-and-kyverno-)
- 🧑‍💻 [Hands On](#hands-on)
- 👀 [References](#references)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

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
| Metrics exposed                             	|      ✓     	|    X    	|
| OpenAPI validation schema (kubectl explain) 	|      X     	|    ✓    	|
| High Availability                           	|      ✓     	|    X    	|
| API object lookup                           	|      ✓     	|    ✓*   	|
| CLI with test ability                       	|     ✓**    	|    ✓    	|
| Policy audit ability                        	|      ✓     	|    ✓    	|

`* Alpha status`
`** Separate CLI`

> Credit: https://kind-brown-cfb734.netlify.app/post/2021-02/kubernetes-policy-comparison-opa-gatekeeper-vs-kyverno/

In my opinion, the best advantages of using Kyverno are no need to learn another policy language and the OpenAPI validation schema support that we can use via kubectl explain command. On the other hand side OPA Gatekeeper supports HA set up and also, there are lots of tools developed around the Rego language to help us to write and test our policies such as [conftest](https://github.com/instrumenta/conftest), [konstraint](https://github.com/plexsystems/konstraint) and this is a big plus in my opinion. These are the tools that we can use to implement `Policy-as-Code Pipeline`. Another advantage of using OPA Gatekeeper, therese are lots of libraries that includes ready to use policies written for us such as [gatekeeper-library](https://github.com/open-policy-agent/gatekeeper-library), [konstraint-examples](https://github.com/plexsystems/konstraint/tree/main/examples) and [raspbernetes-policies](https://github.com/raspbernetes/k8s-security-policies/tree/master/policies).

# Hands On

# References
* https://medium.com/trendyol-tech/enforce-organizational-policies-and-security-best-practices-to-your-kubernetes-clusters-by-using-dfc085528e07
* https://www.velotio.com/engineering-blog/deploy-opa-on-kubernetes
* https://engineering.mercari.com/en/blog/entry/20201222-enhance-kubernetes-security-with-opa-gatekeeper/
* https://docs.hashicorp.com/sentinel/concepts/policy-as-code
* https://www.magalix.com/blog/policy-as-code-for-kubernetes
* https://betterprogramming.pub/policy-as-code-on-kubernetes-with-kyverno-b144749f144
* https://itnext.io/fitness-validation-for-your-kubernetes-apps-policy-as-code-7fad698e7dec
