# Amlen Operator

The ansible operator for deploying Amlen systems. Currently under development.

## Quick Start

The operator requires a certificate manager such as cert-manager which can be deployed via:

kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.8.0/cert-manager.yaml

Assuming you are using an image that already exists you just need to install and deploy it

```
export IMG=quay.io/ianboden/amlen-operator:v1.0.331
make install
make deploy
```

That will deploy the operator into the amlen-system

In config/samples are the yaml files need to deploy a simple setup start with a new project and the deploy the samples:

```
oc new-project amlen
oc apply -f config/samples/simple_password.yaml
oc apply -f config/samples/selfsigned.yaml
oc apply -f config/samples/config.yaml
oc apply -f config/samples/amlen_v1_amlen.yaml
```

This will deploy 2 HA pairs called amlen-0 and amlen-1 into the namespace. amlen-0 will have login details set to sysadmin:passw0rd
while amlen-1 will use a random password stored in the secret amlen-1-password. Both will have created certificates stored in amlen-0-cert-internal 
and amlen-1-cert-internal. (passwords and certificates are mounted in /secrets/ on the amlen pods)


## Default values

In roles/amlen/defaults/main.yml are the default values that are used, these can all be overriden in the Amlen CustomResourceDefinition

```
_amlen_state: present
_amlen_namespace: amlen
_amlen_name: amlen

_amlen_volume_size: 10Gi
_amlen_memory_request: 3Gi
_amlen_memory_limit: 3Gi
_amlen_cpu_request: 200m
_amlen_cpu_limit: 500m
_amlen_wait_for_init: true
_amlen_image: quay.io/ianboden/amlen
_amlen_image_tag: latest
```

These defaults are suitable for creating demo systems only volume size, memory and cpu limits should be sized correctly for production systems

## Limitations

This is still in development and so is mainly focused on installation as such there are a number of limitations:

* Reducing the size does not result in the number of StatefulSets being reduced, they must be removed manually
* Cleanup of PVCs must be done manually
* Updates of Amlen (either by changing the _amlen_image_tag in the CRD or issuing a roll out of the statefulset) is likely to cause double disconnects in HA pairs. After a fresh install instance 0 in the pair will be primary but a subsequent update will make instance 1 to be primary, if instance 1 is the primary then an update will cause double disconnects due to always updating instance 1 first which will mean devices failover to instance 0 then fail back when instance 0 is updated.
* Changing the admin password is dangerous, in theory you can update the secret to the new password then run the rest API command to change the admin password then issue a rolling restart of the statefulset, in practice it may well fall over as the readiness script will fail during this process.

## Testing

A basic molecule test has been written that requires openshift (it uses openshift's ability to expose ports for the test to run).

The test uses some of the sample files to create 2 HA pairs (4 pods in total) using self signed certificates with a config that requires clients to use certificates. It then checks it's possible to publish and subscribe to the second HA pair.

## Samples

### amlen_v1_amlen.yaml

Creates 2 HA pair systems using a simple config and certificate

### _v1alpha_amlen.yaml

Same as amlen_v1_amlen but will use the osdk-test namespace for use in molecule testing

### config.yaml

Creates a simple config based on the default config but requires clients to use certificates for connection

### self_signed.yaml

Create a ClusterIssuer that produces self signed certificates

### simple_password.yaml

An example of how to pre-set the admin password for a system given the system name (the name that matches the stateful set that is created) and the namespace.
