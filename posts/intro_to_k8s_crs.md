
# How To: Kubernetes CRs

Authored by: Akira Wong

Created on: 08/27/2021

Updated on: 09/01/2021

## Pre-requisites

This document assumes you have some background knowledge of Kubernetes & etcd and Golang

## Introduction

Custom Resources Definitions(CRDs) extend the functionality of a Kubernetes Cluster and integrate directly with kube-apiserver and etcd. Defined with YAML files, developers can easily create and register them with a Kubernetes cluster and deploy custom Controllers & Webhooks to drive business logic in a cloud-native way.

  

By themselves, Custom Resources(CRs) are simply data objects stored in the cluster’s etcd database. When paired with Controllers, developers can implement declarative APIs that use the CRs to define a desired state, and utilize the Controllers to reconcile that state with reality.

Learn more about CRDs from the [Kubernetes Documentation](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/)

  

A good place to start writing CRDs and Controllers is with Kubebuilder and Golang

## Kubebuilder

With Kubebuilder, you:

-   Define CRDs as go structs with annotations that define behavior like optional fields and enums
    

Example:
```
// Animal CRs allow us to store Animal instances in etcd

type  Animal  struct {

	// Name defines the name of the animal

	// +kubebuilder:validation:Pattern="^[a-z0-9]([-a-z0-9]*$"

	Name string  `json:"name"`

	// Specifies the current state of the Animal

	// Allowed values are:

	// "INDOOR": Animal is indoor

	// "OUTDOOR": Animal is outdoor

	DesiredState DesiredState `json:"desiredState"`

	// CurrentOwnerRef references the owner currently assigned for the Animal

	CurrentOwnerRef ResourceReference `json:"currentOwnerRef,omitempty"`

}
```
  

Kubebuilder generates:

-   Kubernetes CRD yaml files
    
-   Apply-able Controllers and Webhooks and Kubernetes Service Accounts, Certificates, and Secrets for them
    
-   Scaffolding for testing with envtest and Ginkgo!
    

A fantastic reference is the [Kubebuilder online book](https://book.kubebuilder.io/quick-start.html)



----------

## Tips and Tricks

### Resource Spec vs Status

When defining CRs for the first time, you decide between fields in spec and status.

Spec generally reflects desired state, and is set by the CR user.

Status reflects current state and is updated by Reconcilers- running processes in the cluster responsible for acting to reconcile CR current with desired state.

  

### Resource Status Conditions

Conditions also can be added to a CR’s status section. Status Conditions are made up of string type keys like “Ready”, “Started” and have status values of “True”, “False”, or “Unknown”. They can be used to report a CR’s status to other Controllers in order to centralize logic for detecting if certain conditions are met. For example, a CR Controller can set a CR’s “Ready” condition to “True” when the number of desired instances matches the number of running instances. This makes it so other Controllers in the Cluster do not need to include logic for counting and determining if the CR is ready. Additionally, Conditions can be used to drive control logic within a Controller itself, so it only performs actions like restarting a CR when certain Conditions are set.

### Eventual Consistency

Reconcilers/Controllers operate on an "eventual consistency" model where it takes time for changes to propagate throughout the system. A Controller constantly attempts to reconcile a Reource’s last-known state and last-known desired state, without worrying about the past.

Implementation-wise Controllers have a "Reconcile" function that is called whenever a CR of the type the Controller is watching is Created, Updated, or Deleted. It is up to the Controller to decide based on the current vs desired state to take actions to reconcile and move towards the desired state. If there is an error in reconciliation, the Controller will automatically re-queue the Reconcile request so it will hopefully be more stable and have more information to process it then.

### Admission Webhooks

Admission Webhooks, validating and mutating operate at a different level than Controllers. They take the form of HTTP callback functions that “intercept” Custom Resource CRUD requests and do something with them. Validating Webhooks are called by the Kubernetes API whenever a CR is submitted- they examine the request and perform additional validation, making sure the request is well-formed in accordance to the developer's specification before accepting or rejecting the request. One example application of Validating Webhooks is to ensure uniqueness of certain fields. Mutating Webhooks are similar in that they are registered to be called automatically by the Kubernetes API, but differ in that they directly modify the request sent, to do things like add default values, and append labels automatically. Both types of webhooks are written by the CR developer and must be registered and deployed to the Kubernetes Cluster as images in some form of deployment.

  

----------

  

## Useful Links

-   Kubernetes CRD Documentation: [https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/)
    
-   Kubernetes Documentation, Extending the Kubernetes API:  
    [https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/)
    
-   Kubernetes Documentation, Admission Webhooks: [https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/#what-are-admission-webhooks](https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/#what-are-admission-webhooks)
    
-   Kubebuilder book: https://book.kubebuilder.io/
    
-   What the heck are Conditions: [https://maelvls.dev/kubernetes-conditions/](https://maelvls.dev/kubernetes-conditions/)