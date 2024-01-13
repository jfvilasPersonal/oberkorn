# Oberkorn installation
Installing Oberkorn is very simple and straightforward.

The process includes:
  - Creating Custom Resource Definitions (CRD) on your kubernetes cluster.
  - Deploying the Oberkorn controller.

Once everything is ok, you can start deploying Oberkorn authorizators to protect your applications.

## Setup
**Step 1**. Create the CRD.
You can download and review the CRD or you can just apply it to your cluster. **Applying a CRD is not harmful**.

```sh
kubectl apply -f https://raw.githubusercontent.com/jfvilasPersonal/obk-controller/main/crd/crd.yaml
```

This will create a CRD which you will use to create your authorizators.

**Step 2**. Deploy the controller.
The controller is the software component in charge of managing your authorizators. It listens for kubernetes events like ADD, DELETE o MODIFY anytime you create, remove or modify one authorizator. To deploy the controller you just need to apply a YAML that contains several definitions.

```sh
kubectl apply -f https://raw.githubusercontent.com/jfvilasPersonal/obk-controller/main/crd/controller.yaml
```

The controller YAML contains several kubernetes resources:

 - A *Deployment*, which launchs a pod based on the Oberkorn controller image ('obk-controller' available on docker hub).
 - A *Service Account*. The controller needs some permission to manage resources inside the cluster, like creating services or deployments, so we need a service account that we assign to the deployment.
 - A *Cluster Role*. The permissions needed by the controller are defined inside a cluster role named 'obk-controller-sa'.
 - Finally, a *Cluster Role Binding* named 'obk-controller-crb' assigns the cluster role to the created service account.

You can check whether the controller is ready by examining controller stdout (the logs of the pod) and searching for the message "Oberkorn Controller is watching events...".

## Next step
Everyting is ready! Next you should check if there is any special configuration that you must perform depending on the ingress controller you are using. Continue to [Ingress configuration](/ingress-configuration).
