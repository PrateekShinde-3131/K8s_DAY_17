
# php-apache Deployment with Horizontal Pod Autoscaler

This guide provides steps to deploy a `php-apache` application on Kubernetes and set up a Horizontal Pod Autoscaler (HPA) to automatically scale the application based on CPU utilization.

## Prerequisites

- Kubernetes cluster set up and running.
- `kubectl` CLI installed and configured to communicate with your cluster.

## Step 1: Create the Deployment

The `php-apache` deployment will use the `registry.k8s.io/hpa-example` image and be exposed via a Kubernetes Service.

1. Save the following YAML to `php-apache-deployment.yaml`:

   ```
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: php-apache
   spec:
     selector:
       matchLabels:
         run: php-apache
     template:
       metadata:
         labels:
           run: php-apache
       spec:
         containers:
         - name: php-apache
           image: registry.k8s.io/hpa-example
           ports:
           - containerPort: 80
           resources:
             limits:
               cpu: 500m
             requests:
               cpu: 200m
   ---
   apiVersion: v1
   kind: Service
   metadata:
     name: php-apache
     labels:
       run: php-apache
   spec:
     ports:
     - port: 80
     selector:
       run: php-apache
   ```

2. Apply the deployment and service:

   ```
   kubectl apply -f php-apache-deployment.yaml
   ```

## Step 2: Create the Horizontal Pod Autoscaler (HPA)

The HPA will automatically scale the `php-apache` deployment based on CPU utilization.

1. Save the following YAML to `php-apache-hpa.yaml`:

   ```
   apiVersion: autoscaling/v2
   kind: HorizontalPodAutoscaler
   metadata:
     name: php-apache
   spec:
     scaleTargetRef:
       apiVersion: apps/v1
       kind: Deployment
       name: php-apache
     minReplicas: 1
     maxReplicas: 10
     metrics:
     - type: Resource
       resource:
         name: cpu
         target:
           type: Utilization
           averageUtilization: 50
   ```

2. Apply the HPA:

   ```
   kubectl apply -f php-apache-hpa.yaml
   ```

## Step 3: Simulate Load on the Application

To observe the autoscaler in action, you can generate load on the application using a busybox container.

1. Run the following command to start a load generator:

   ```
   kubectl run -i --tty load-generator --rm --image=busybox:1.28 --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache; done"
   ```

This will continuously send requests to the `php-apache` service, increasing the CPU load.

## Step 4: Monitor the Horizontal Pod Autoscaler

To monitor the HPA and see how it adjusts the number of replicas based on CPU utilization, run:

```
kubectl get hpa php-apache --watch
```

You will see output showing how the number of replicas changes based on the observed CPU usage.

## Clean Up

To clean up all the resources created during this tutorial, run:

```
kubectl delete deployment php-apache
kubectl delete service php-apache
kubectl delete hpa php-apache
```

## Conclusion

You have successfully deployed a `php-apache` application and set up a Horizontal Pod Autoscaler to scale it based on CPU utilization. The HPA ensures that the application can handle varying loads by automatically adjusting the number of running pods.
