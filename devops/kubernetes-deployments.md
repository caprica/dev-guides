# Kubernetes Deployments

[Kubernetes](https://kubernetes.io) is used to orchestrate deployments of applications and services.

This guide assumes that `kubectl` is already and the AWS command-line application (version 2) `aws` are already installed and working.

_Installation structions for kubectl are [here](https://kubernetes.io/docs/tasks/tools/install-kubectl), and for aws [here](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html)._

The kubectl command can be used to deploy applications via Docker images to the cloud.

The cloud environment must already have been provisioned, Kubernetes will not create the cluster and related resources.

Kubernetes knows which cloud environment to deploy to based on the contents of the `~/.kube/config` file. The contents of this file will be different for different target environments.

## Using kubectl or aws behind a corporate proxy

It is likely necessary to set the `HTTP_PROXY` environment variable when running kubectl or aws commands.

This can be provided manually for every command, for example:

```
HTTP_PROXY=http://corporate-proxy:8080 kubectl get pods
```

It is however much more convenient to add that environment variable to the `~/.bash_profile` file so it is automatically available in every new terminal session.

Without setting this, the commands will likely timeout with no explanation.

## Configuration for AWS Cloud

Assuming a suitable cluster has already been provisioned in AWS, an aws command is used to generate the necessary Kubernetes configuration.

```
aws eks --region eu-west-1 update-kubeconfig --name proof-of-concept-cluster
```

In this example, the cluster name `proof-of-concept-cluster` matches the name of a cluster already provisioned in AWS.

Executing the command will update the `~/.kube/config` file with the AWS cluster details.

## Apply Kubernetes configuration files

The example project uses the following Kubernetes configuration files:

 * proof-of-concept-secret.yaml
 * proof-of-concept-deployment.yaml
 * proof-of-concept-service.yaml

The 'secret' file contains various named values required to perform or configure the deployment (for example AWS access keys may be stored here).

The 'deployment' file describes the desired container structure, including the Docker image to use for the deployment.

The 'service' file describes the service itself, how it will be exposed from the cloud.

To use these configuration files it is necessary to "apply" them, in these example commands assume the files are in the current directory:

```
kubectl apply -f ./proof-of-concept-secret.yaml
kubectl apply -f ./proof-of-concept-deployment.yaml
kubectl apply -f ./proof-of-concent-service.yaml
```

The configuration files may reference values in other files, it is very important that the files are 'stitched together' correctly by using corresponding names.

If the configuration files were all correct, the application will be deployed to the clould environment and available for use - nothing else is required.

_These configuration files are cloud-vendor **neutral**, the particular clould platform is specified by the contents of the `~/.kube/config` file._

Reference documentation is [here](https://kubernetes.io/docs/concepts/configuration).

## Using kubectl

The kubectl command provides many functions to maintain the deployment.

Some basic use-cases follow here.

### Basic status

The Kubernetes deployment unit is a `pod`, to show the current status of the pods:

```
kubectl get pods
```

Example session:

```
mark@serenity:~$ kubectl get pods
NAME                                           READY   STATUS    RESTARTS   AGE
proof-of-concept-deployment-757f7677c6-slhv5   1/1     Running   0          19h
mark@serenity:~$
```

### Detailed status

Use the `describe` command to get more detail:

```
kubectl describe pods
```

Or to describe a particular pod:

```
kubectl describe pods proof-of-concept-deployment-757f7677c6-slhv5
```

This may be particularly useful if a deployment failed, the error is likely visible in this output.

### Getting and describing resources

As well as pods, the `get` and `describe` commands can be used with other resources:

#### Services

```
mark@serenity:~$ kubectl get services
NAME                       TYPE           CLUSTER-IP    EXTERNAL-IP       PORT(S)          AGE
kubernetes                 ClusterIP      10.100.0.1    <none>            443/TCP          21h
proof-of-concept-service   LoadBalancer   10.100.38.138 example.com       8080:32265/TCP   19h
mark@serenity:~$
```

Getting, or describing, a service can be useful to determine the external server/IP - here `example.com` is used.

The external IP is that of the load-balancer, and this is what would be used in e.g. a browser or a web-service invocation.

```
mark@serenity:~$ kubectl describe services/proof-of-concept-service
Name:                     proof-of-concept-service
Namespace:                default
Labels:                   <none>
Annotations:              kubectl.kubernetes.io/last-applied-configuration:
                            {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"name":"proof-of-concept-service","namespace":"default"},"spec":{"ports":...
Selector:                 app=proof-of-concept
Type:                     LoadBalancer
IP:                       10.100.38.138
LoadBalancer Ingress:     example.com
Port:                     http  8080/TCP
TargetPort:               8080/TCP
NodePort:                 http  32265/TCP
Endpoints:                192.168.74.185:8080
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
mark@serenity:~$
```

#### Deployments

To get the list of deployments and a high-level status:

```
mark@serenity:~$ kubectl get deployments
NAME                          READY   UP-TO-DATE   AVAILABLE   AGE
proof-of-concept-deployment   1/1     1            1           19h
mark@serenity:~$
```

Describe the deployment to show more details:

```
mark@serenity:~$ kubectl describe deployments/proof-of-concept-deployment
Name:                   proof-of-concept-deployment
Namespace:              default
CreationTimestamp:      Mon, 02 Mar 2020 15:05:33 +0000
Labels:                 app=proof-of-concept
Annotations:            deployment.kubernetes.io/revision: 2
                        kubectl.kubernetes.io/last-applied-configuration:
                          {"apiVersion":"apps/v1beta1","kind":"Deployment","metadata":{"annotations":{},"name":"proof-of-concept-deployment","namespace":"default"},...
Selector:               app=proof-of-concept
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=proof-of-concept
  Containers:
   poc:
    Image:      629587629376.dkr.ecr.eu-west-1.amazonaws.com/proof-of-concept:b54bb17
    Port:       8080/TCP
    Host Port:  0/TCP
    Environment:
      AWS_ACCESS_KEY:  <set to the key 'awsAccessKey' in secret 'proof-of-concept-secret'>  Optional: false
      AWS_SECRET_KEY:  <set to the key 'awsSecretKey' in secret 'proof-of-concept-secret'>  Optional: false
    Mounts:            <none>
  Volumes:             <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   proof-of-concept-deployment-757f7677c6 (1/1 replicas created)
Events:          <none>
mark@serenity:~$
```

Among other things, this output shows the Docker image that is used by the deployment.

#### Secrets

To verify that secrets have been configured correctly:

```
mark@serenity:~$ kubectl get secrets
NAME                      TYPE                                  DATA   AGE
default-token-t9gxq       kubernetes.io/service-account-token   3      21h
proof-of-concept-secret   Opaque                                2      19h
mark@serenity:~$
```

This output does not show the secret values themselves, but it can be seen that the secrets exist and that there are two configured secret values.

Describing shows some more details:

```
mark@serenity:~$ kubectl describe secrets proof-of-concept-secret
Name:         proof-of-concept-secret
Namespace:    default
Labels:       <none>
Annotations:
Type:         Opaque

Data
====
awsAccessKey:  21 bytes
awsSecretKey:  41 bytes
mark@serenity:~$
```

### Deployment rollout status

For a high-level summary on the state of a deployment:

```
mark@serenity:~$ kubectl rollout status deployments/proof-of-concept-deployment
deployment "proof-of-concept-deployment" successfully rolled out
mark@serenity:~$
```

A `-w` switch can be added to the command to 'watch' the deployment until it is done:

```
kubectl rollout status deployments/proof-of-concept-deployment -w
```

### Other commands

The preceding sections show examples of the basic, most immediately useful, functionality provided by kubectl. There are numerous more commands available.

Reference documentation is [here](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands).

## Accessing the deployed service

During development there may not be an appropriate DNS record in place for the service and so the default name provided by the cloud platform will be used.

As previously described, the kubectl command can be used to get the srevice details and this will include the external IP to use.

This information can also be obtained from the AWS console - select `EC2` then `Load Balancers` and locate the desired DNS name from the table of load-balancers.

Once the DNS name is known, it can be used in a web browser or by web-service invocations.

## Manually undeploying, or scaling up, the service

To undeploy the service, in the deployment configuration file set the number of replicas to zero.

Similarly, to change the number of deployed instances set that number of replicas to the desired value.

After making the change to the configuration file, simply `apply` it again (as described ealier) and the new demand will be automatically applied to the associated cluster.

## Alternatives for secrets

Using Kubernetes to manage secrets is just one option.

Clould platforms usually provide their own "vault" for storing and accessing application secrets and this may be more appropriate than using Kubernetes secrets. For example in AWS this would be the [AWS Secrets Manager](https://aws.amazon.com/secrets-manager).
