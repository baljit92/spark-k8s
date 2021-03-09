We will bring up a Jupyter notebook in Kubernetes and run a Spark application in client mode. We will also use [sparkmonitor](https://github.com/swan-cern/jupyter-extensions/tree/master/SparkMonitor) widget for visualization. 

### Jupyter installation:

Our setup contains two images:

1. Spark image — used for spinning up Spark executors.
2. Jupyter notebook image — used for Jupyter notebook and Spark driver.

### Setup instructions:

1. Create an cluster (AKS, in our case). The command below is just for an example cluster. It can be customized based on the user needs. The important option to include is the `--enable-cluster-autoscaler` to make sure the scaling is Automatic and not Manual.

```
# Now create the AKS cluster and enable the cluster autoscaler
az aks create \
  --resource-group myResourceGroup \
  --name myAKSCluster \
  --node-count 1 \
  --vm-set-type VirtualMachineScaleSets \
  --load-balancer-sku standard \
  --enable-cluster-autoscaler \
  --min-count 1 \
  --max-count 3
```

Next, create a new dedicated namespace for spark, install the relevant Kubernetes resources and expose Jupyter’s port:

```
kubectl create ns spark
kubectl apply -n spark -f jupyter.yaml
kubectl port-forward -n spark service/jupyter 8888:8888
```
Note that a dedicated namespace has several benefits:

1. **Security** — as Spark requires permissions to create/delete pods etc. it’s better to limit those permissions to a specific namespace.
2. **Observability** — Spark might spawn a lot of executor pods so it might be easier to track those if they are isolated in a separate namespace. On the other hand, you don’t want to miss any other application pods between all of those executor pods.

That’s it! Now open your browser and go to http://127.0.0.1:8888 and run our first Spark application. You can use the notebooks included as an example.

1. `spark_application.ipynb` has the Spark configuration that will be used to launch the Spark session
2. `demo.ipynb` just reads a csv file from s3 storage

Note: In order to change the storageClassName; execute `kubectl get storageclass --all-namespaces` and choose the type of disk and enter the Name value in the `jupyter.yaml` file.

Interesting repo for optimizing Spark on EKS: https://github.com/aws-samples/eks-spark-benchmark

### Reference
1. https://kublr.com/blog/running-spark-with-jupyter-notebook-hdfs-on-kubernetes/
2. https://towardsdatascience.com/jupyter-notebook-spark-on-kubernetes-880af7e06351