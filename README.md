# n +/- CRD compatibility proposal

## Problem

Switching a CRD from v1beta1 to v1 with v1 as stored version breaks the abality to automatically downgrade.

## Proposal

Make the migration in two steps:

1. 1.17: Add the new version as served version and keep the old version as stored version.
2. 1.18: Switch the stored version to the new version.

## Lab test


### Emulate version 1.16 - v1beta1
```console
$ k apply -f pkg.crossplane.io_functionrevisions_1.16_beta_only.yaml
customresourcedefinition.apiextensions.k8s.io/functionrevisions.pkg.crossplane.io created

$ k apply -f obj.yaml
functionrevision.pkg.crossplane.io/function-go-templating-7f94b920392a created
```


### Emulate version 1.17 - v1beta1 and v1 served, v1beta1 stored
```console
$ k apply -f pkg.crossplane.io_functionrevisions_1.17_beta_and_v1_step_1_beta_served.yaml
customresourcedefinition.apiextensions.k8s.io/functionrevisions.pkg.crossplane.io configured
$ k get -f obj.yaml
NAME                                  HEALTHY   REVISION   IMAGE                                                              STATE    DEP-FOUND   DEP-INSTALLED   AGE
function-go-templating-7f94b920392a             2          xpkg.upbound.io/crossplane-contrib/function-go-templating:v0.5.0   Active                               12s
```

### Emulate downgrade from 1.17 to 1.16
```console
$ k apply -f pkg.crossplane.io_functionrevisions_1.16_beta_only.yaml
customresourcedefinition.apiextensions.k8s.io/functionrevisions.pkg.crossplane.io configured
```

### Emulate upgrade from 1.16 to 1.17 and then to 1.18
```console
$ k apply -f pkg.crossplane.io_functionrevisions_1.17_beta_and_v1_step_1_beta_served.yaml
customresourcedefinition.apiextensions.k8s.io/functionrevisions.pkg.crossplane.io configured
$ k apply -f pkg.crossplane.io_functionrevisions_1.18_beta_and_v1_step_2_v1_served.yaml
customresourcedefinition.apiextensions.k8s.io/functionrevisions.pkg.crossplane.io configured
$ k get -f obj.yaml
NAME                                  HEALTHY   REVISION   IMAGE                                                              STATE    DEP-FOUND   DEP-INSTALLED   AGE
function-go-templating-7f94b920392a             2          xpkg.upbound.io/crossplane-contrib/function-go-templating:v0.5.0   Active                               35s
```

### Emulta downgrade to 1.17
```console
$ k apply -f pkg.crossplane.io_functionrevisions_1.17_beta_and_v1_step_1_beta_served.yaml
customresourcedefinition.apiextensions.k8s.io/functionrevisions.pkg.crossplane.io configured
```

### Test invalid downgrade from 1.18 to 1.16 (n-2)
```console
$ k apply -f pkg.crossplane.io_functionrevisions_1.16_beta_only.yaml
The CustomResourceDefinition "functionrevisions.pkg.crossplane.io" is invalid: status.storedVersions[1]: Invalid value: "v1": must appear in spec.versions
```
