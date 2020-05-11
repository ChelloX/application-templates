# Application Templates

This project contains OpenShift v3 application templates which support
applications based on Red Hat Fuse.

## Article on how to get started with this OpenShift template

See http://www.opensourcerers.org/first-fuse-application-on-openshift-v3-1/

## Configure Authentication to the Red Hat Container Registry

Before you can import and use the Red Hat Fuse OpenShift Image Streams, you must first configure authentication to the Red Hat container registry.
This may have already been set up for you by your cluster administrator. If not, follow the instructions bellow.

Within the OpenShift project where you'll be installing the Red Hat Fuse Image Streams, create a docker-registry secret using credentials for either a
Red Hat Developer Program account or Red Hat Customer Portal account. For example:

```
oc create secret docker-registry imagestreamsecret \
  --docker-server=registry.redhat.io \
  --docker-username=CUSTOMER_PORTAL_USERNAME \
  --docker-password=CUSTOMER_PORTAL_PASSWORD \
  --docker-email=EMAIL_ADDRESS
```

If you don’t want to use your Red Hat account’s username and password to create the secret, it is recommended to [create an authentication token](https://access.redhat.com/RegistryAuthentication#registry-service-accounts-for-shared-environments-4) using a [registry service account](https://access.redhat.com/terms-based-registry/).

For further information see:

* [Red Hat Container Registry Authentication](https://access.redhat.com/RegistryAuthentication)
* [Accessing and Configuring the Red Hat Registry](https://docs.openshift.com/container-platform/3.11/install_config/configuring_red_hat_registry.html)

## Install Image Streams

The `fis-image-streams.json` file contains __ImageStream__ definitions for Red Hat Fuse on OpenShift. This will need to be installed within
the `openshift` namespace (`oc create -f fis-image-streams.json -n openshift`) before using any of the quickstart templates.

## Example

The easiest way to use the quickstart templates is to install them in your project

```
$ oc create -n openshift -f fis-image-streams.json
$ oc create -n myproject -f quickstart-template.json
```

After installing templates, you should see the quickstarts in the OpenShift catalog. Select one, and you will be prompted to
provide template parameters.

Alternatively, you can install the template and create a new application  from the command line:
```
$ oc create -n openshift -f fis-image-streams.json
$ oc process -n yourproject -f quickstart-template.json -v <template parameters> | oc create -n yourproject -f -
```

To make templates available to all users, install them into the `openshift` namespace:
```
$ oc create -n openshift -f quickstart-template.json
```

## Fuse Console

The Red Hat Fuse console eases the discovery and management of Fuse applications deployed on OpenShift.

You can run the following instructions to deploy the Fuse console on your OpenShift cluster.
There exist different OpenShift templates to choose from, depending on the following characteristics:

| Template | Descripton |
| -------- | ---------- |
| [fis-console-cluster-template.json](https://raw.githubusercontent.com/jboss-fuse/application-templates/master/fis-console-cluster-template.json) | Use an OAuth client that requires the `cluster-admin` role to be created. The Fuse console can discover and connect to Fuse applications deployed across multiple namespaces / projects. |
| [fuse-console-cluster-os4.json](https://raw.githubusercontent.com/jboss-fuse/application-templates/master/fuse-console-cluster-os4.json) | Same as `fis-console-cluster-template.json`, to be used for OpenShift 4. By default, this requires the generation of a client certificate, signed with the [service signing certificate][service-signing-certificate] authority, prior to the deployment. See [OpenShift 4](#openshift-4) section for more information. |
| [fis-console-namespace-template.json](https://raw.githubusercontent.com/jboss-fuse/application-templates/master/fis-console-namespace-template.json) | Use a service account as OAuth client, which only requires `admin` role in a project to be created. This restricts the Fuse console access to this single project, and as such acts as a single tenant deployment. |
| [fuse-console-namespace-os4.json](https://raw.githubusercontent.com/jboss-fuse/application-templates/master/fuse-console-namespace-os4.json) | Same as `fis-console-namespace-template.json`, to be used for OpenShift 4. By default, this requires the generation of a client certificate, signed with the [service signing certificate][service-signing-certificate] authority, prior to the deployment. See [OpenShift 4](#openshift-4) section for more information. |
| [fuse-console-cluster-rbac.yml](https://raw.githubusercontent.com/jboss-fuse/application-templates/master/fuse-console-cluster-rbac.yml) | Same as `fuse-console-cluster-os4.json`, with configurable RBAC for Jolokia requests authorization. See the [RBAC](#rbac) section for more information. |
| [fuse-console-namespace-rbac.yml](https://raw.githubusercontent.com/jboss-fuse/application-templates/master/fuse-console-namespace-rbac.yml) | Same as `fuse-console-namespace-os4.json`, with configurable RBAC for Jolokia requests authorization. See the [RBAC](#rbac) section for more information. |

[service-signing-certificate]: https://docs.openshift.com/container-platform/4.1/authentication/certificates/service-serving-certificate.html

For example, to install the Fuse console template, execute the following command:

```sh
$ oc create -n myproject -f fis-console-namespace-template.json
```

Then, you should be able to see the template after navigating to _Add to Project > Select from Project_ in your project.

Or, if you prefer the command line:

```sh
$ oc new-app -n myproject -f fis-console-namespace-template.json \
  -p ROUTE_HOSTNAME=<HOST>
```

Note that the `ROUTE_HOSTNAME` parameter can be omitted when using the `fis-console-namespace-template` template.
In that case, OpenShift automatically generates one for you.

You can obtain more information about the template parameters, by executing the following command:

```
$ oc process --parameters -f fis-console-namespace-template.json
NAME                     DESCRIPTION                                                                    VALUE
APP_NAME                 The name assigned to the application.                                          fuse70-console
APP_VERSION              The application version.                                                       1.0
IMAGE_STREAM_NAMESPACE   Namespace in which the Fuse ImageStreams are installed. These ImageStreams
                         are normally installed in the openshift namespace. You should only need to
                         modify this if you've installed the ImageStreams in a different
                         namespace/project.                                                             openshift
ROUTE_HOSTNAME           The externally-reachable host name that routes to the Red Hat Fuse console
                         service
CPU_REQUEST              The amount of CPU to request.                                                  0.2
MEMORY_REQUEST           The amount of memory required for the container to run.                        32Mi
CPU_LIMIT                The amount of CPU the container is limited to use.                             1.0
MEMORY_LIMIT             The amount of memory the container is limited to use.                          32Mi
```

You can obtain the status of your deployment, by running:

```sh
$ oc status
In project myproject on server https://192.168.64.12:8443

https://fuse-console.192.168.64.12.nip.io (redirects) (svc/fuse70-console-service)
  dc/fuse70-console deploys openshift/jboss-fuse70-console:1.0
    deployment #1 deployed 2 minutes ago - 1 pod
```

Open the route URL displayed above from your Web browser to access the Fuse console.

### OpenShift 4

To secure the communication between the Fuse Console and the Jolokia agents, a client certificate must be generated and mounted into the Fuse Console pod with a secret, to be used for TLS client authentication. This client certificate must be signed using the [service signing certificate][service-signing-certificate] authority private key.

Here are the steps to be performed prior to the deployment:

1. First, retrieve the service signing certificate authority keys, by executing the following commmands as a _cluster-admin_ user:
    ```sh
    # The CA certificate
    $ oc get secrets/signing-key -n openshift-service-ca -o "jsonpath={.data['tls\.crt']}" | base64 --decode > ca.crt
    # The CA private key
    $ oc get secrets/signing-key -n openshift-service-ca -o "jsonpath={.data['tls\.key']}" | base64 --decode > ca.key
    ```

2. Then, generate the client certificate, as documented in [Kubernetes certificates administration](https://kubernetes.io/docs/concepts/cluster-administration/certificates/), using either `easyrsa`, `openssl`, or `cfssl`, e.g., using `openssl`:
    ```sh
    # Generate the private key
    $ openssl genrsa -out server.key 2048
    # Write the CSR config file
    $ cat <<EOT >> csr.conf
      [ req ]
      default_bits = 2048
      prompt = no
      default_md = sha256
      distinguished_name = dn

      [ dn ]
      CN = fuse-console.fuse.svc

      [ v3_ext ]
      authorityKeyIdentifier=keyid,issuer:always
      keyUsage=keyEncipherment,dataEncipherment,digitalSignature
      extendedKeyUsage=serverAuth,clientAuth
    EOT
    # Generate the CSR
    $ openssl req -new -key server.key -out server.csr -config csr.conf
    # Issue the signed certificate
    $ openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt -days 10000 -extensions v3_ext -extfile csr.conf
    ```

3. Finally, you can create the secret to be mounted in the Fuse Console, from the generated certificate:
   ```sh
   $ oc create secret tls ${APP_NAME}-tls-proxying --cert server.crt --key server.key
   ```

Note that `CN=fuse-console.fuse.svc` must be trusted by the Jolokia agents, for which client certification authentication is enabled. See the `clientPrincipal` parameter from the [Jolokia agent configuration options](https://jolokia.org/reference/html/agents.html#agent-jvm-config).

You can then proceed with the [deployment](#deployment).

### RBAC

#### Configuration

The `fuse-console-cluster-rbac.yml` and `fuse-console-namespace-rbac.yml` templates create a _ConfigMap_, that contains the configuration file used to define the roles allowed for MBean operations.
This _ConfigMap_ is mounted into the Fuse console container, and the `HAWTIO_ONLINE_RBAC_ACL` environment variable is used to pass the configuration file path to the server.
If that environment variable is not set, RBAC support is disabled, and only users granted the `update` verb on the pod resources are authorized to call MBeans operations.

#### Roles

For the time being, only the `viewer` and `admin` roles are supported.
Once the current invocation is authenticated, these roles are inferred from the permissions the user impersonating the request is granted for the pod hosting the operation being invoked.

A user that's granted the `update` verb on the pod resource is bound to the `admin` role, i.e.:

```sh
$ oc auth can-i update pods/<pod> --as <user>
yes
```

Else, a user granted the `get` verb on the pod resource is bound the `viewer` role, i.e.:

```sh
$ oc auth can-i get pods/<pod> --as <user>
yes
```

Otherwise the user is not bound any roles, i.e.:

```sh
$ oc auth can-i get pods/<pod> --as <user>
no
```

#### ACL

The ACL definition for JMX operations works as follows:

Based on the _ObjectName_ of the JMX MBean, a key composed with the _ObjectName_ domain, optionally followed by the `type` attribute, can be declared, using the convention `<domain>.<type>`.
For example, the `java.lang.Threading` key for the MBean with the _ObjectName_ `java.lang:type=Threading` can be declared.
A more generic key with the domain only can be declared (e.g. `java.lang`).
A `default` top-level key can also be declared.
A key can either be an unordered or ordered map, whose keys can either be string or regexp, and whose values can either be string or array of strings, that represent roles that are allowed to invoke the MBean member.

The default ACL definition can be found in the `${APP_NAME}-rbac` _ConfigMap_ from the `fuse-console-cluster-rbac.yml` and `fuse-console-namespace-rbac.yml` templates.

#### Authorization

The system looks for allowed roles using the following process:

The most specific key is tried first. E.g. for the above example, the `java.lang.Threading` key is looked up first.
If the most specific key does not exist, the domain-only key is looked up, otherwise, the `default` key is looked up.
Using the matching key, the system looks up its map value for:

1. An exact match for the operation invocation, using the operation signature, and the invocation arguments, e.g.:

   `uninstall(java.lang.String)[0]: [] # no roles can perform this operation`

2. A regexp match for the operation invocation, using the operation signature, and the invocation arguments, e.g.:

   `/update\(java\.lang\.String,java\.lang\.String\)\[[1-4]?[0-9],.*\]/: admin`

   Note that, if the value is an ordered map, the iteration order is guaranteed, and the first matching regexp key is selected;

3. An exact match for the operation invocation, using the operation signature, without the invocation arguments, e.g.:

   `delete(java.lang.String): admin`

4. An exact match for the operation invocation, using the operation name, e.g.:

   `dumpStatsAsXml: admin, viewer`

If the key matches the operation invocation, it is used and the process will not look for any other keys. So the most specific key always takes precedence.
Its value is used to match the role that impersonates the request, against the roles that are allowed to invoke the operation.
If the current key does not match, the less specific key is looked up and matched following the steps 1 to 4 above, up until the `default` key.
Otherwise, the operation invocation is denied.


## Prometheus Operator for Fuse

Monitor your Fuse applications and take advantage of the built-in instrumentation with the prometheus-operator.

You can run the following instructions to deploy the prometheus-operator in your Fuse namespace:

First, install the CustomResourceDefinitions necessary for running the prometheus-operator.   The CustomResourceDefinitions only need to be installed once per cluster, so if you are installing the prometheus-operator to multiple namespaces, you only need to run this step once.   Note that you will need to be logged in as a user with cluster admin permissions.

```
$ oc login -u system:admin
$ oc create -f fuse-prometheus-crd.yml
```

Then, install the prometheus-operator to your namespace:

```
$ oc process -f fuse-prometheus-operator.yml  -p NAMESPACE=<YOUR NAMESPACE> | oc create -f -
```

Finally, tell it to monitor your fuse application:

```
$ oc process -f fuse-servicemonitor.yml -p NAMESPACE=<YOUR NAMESPACE> FUSE_SERVICE_NAME=<YOUR FUSE SERVICE> | oc apply -f -
```

Note that the `NAMESPACE` and `FUSE_SERVICE_NAME` parameters must be specified.


## Fuse Apicurito

Design beautiful, functional APIs with zero coding, using a visual designer for OpenAPI documents.

You can run the following instructions to deploy Fuse Apicurito on your OpenShift cluster.
To install the Fuse Apicurito template, execute the following command:

```sh
$ oc create -n myproject -f fuse-apicurito.yml
```

Then, you should be able to see the template after navigating to _Add to Project > Select from Project_ in your project.

Or, if you prefer the command line:

```sh
$ oc new-app --template apicurito -p ROUTE_HOSTNAME=<HOST>
```

Note that the `ROUTE_HOSTNAME` parameter must be specified and set to a hostname that will resolve to your openshift cluster.




