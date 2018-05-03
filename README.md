# k8s-scaling-demo
Demo about K8s' scaling capabilities using GKE

## Usage
### Prior
- Set up GCP account (Gmail)
- Create project
- Set up billing
- Set up GKE API
- Go to [GCP console](https://console.cloud.google.com)
- Open Cloud Shell (small terminal icon upper-left)

### Deployment
1. Spin up a GKE cluster:

   `gcloud container clusters create k8s-demo-cluster --num-nodes 2 --zone europe-west1-b`

1. Build the container and push to Google Cloud Registry:

   `git clone https://github.com/GoogleCloudPlatform/nodejs-docs-samples.git`

   `cd nodejs-docs-samples/containerengine/hello-world/`
   
   `docker build -t gcr.io/$DEVSHELL_PROJECT_ID/hello-node:1.0 .`
   
   `gcloud docker -- push gcr.io/$DEVSHELL_PROJECT_ID/hello-node:1.0`

1. Deploy a 'hello-world' app deployment:

   `kubectl run hello-node --image=gcr.io/$DEVSHELL_PROJECT_ID/hello-node:1.0 --port=8080 --replicas 2`
   
   Check status with `kubectl get deployments` and `kubectl get pods`.
   
1. Create a service and expose it externally through GCP's Load Balancer

   `kubectl expose deployment hello-node --name=hello-node-svc --type=LoadBalancer --port=80 --target-port=8080`
   
   Check status with `kubectl get services` and wait for the external ephimeral IP assignment.
   
   Verify success accesing the service through this external IP, either in your browser or with `curl`.

1. Update the deployment

   Increase the replica number. Go to 'Workloads' in GKE console and copy the yaml code corresponding to the 'hello-node' deployment into 'app-deployment.yaml', modifying the replica number.
   
   `kubectl apply -f app-deployment.yaml`

1. Create a horizontal pod autoscale resource

   With a mininum of 2 and a maximum of 10 replicas:
   
   `kubectl autoscale deployment hello-node --min=2 --max=10 --cpu-percent=50`

### Load test
Install 'httperf' on your system, e. g. `sudo apt-get install httperf`.
Consider using a VM instead of Cloud Shell for more throughput.

Create a sinthetic test load for the app and see the autoscaler in action:

`httperf --server SERVICEIP --num-conns 100 --rate 10 --timeout 1`
   
Play with some loads and see the autoscaler creating and destroying pods:

`watch -n 1 kubectl get pods`

### Clean up
Clean up all resources from GCP's console: deployments, LB, cluster, firewall routes, etc.

## Author
- Marcos Manuel Ortega
- info@indavelopers.com
- GDG Almería organizer
- GDG Cloud Español coorganizer
- 2/may/2018
- For AlmeriaJS meetup

## License
See LICENSE file.
