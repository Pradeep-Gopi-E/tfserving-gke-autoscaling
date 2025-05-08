# 🚀 TensorFlow Serving on GKE with Autoscaling

This project demonstrates how to deploy a TensorFlow model using TensorFlow Serving on Google Kubernetes Engine (GKE) with Horizontal Pod Autoscaler based on CPU utilization. It also includes performance testing using Locust.

---

## 📦 Getting Started

### 1. Clone the Repository in Google Cloud Shell

Start by cloning this repository into your Cloud Shell:

```bash
git clone https://github.com/Pradeep-Gopi-E/tfserving-gke-autoscaling.git
cd tfserving-gke-autoscaling
```

> **Note:** Use a GitHub Personal Access Token (PAT) when prompted for a password if you encounter authentication issues. Learn more: [Creating a personal access token](https://docs.github.com/en/github/authenticating-to-github/creating-a-personal-access-token).

---

## ⚙️ Setup and Requirements

### 2. Set your GCP Project ID and Compute Zone

In Cloud Shell, configure your Google Cloud project and default compute zone. Replace `your-gcp-project-id` with your actual project ID.

```bash
PROJECT_ID=your-gcp-project-id
gcloud config set project $PROJECT_ID
gcloud config set compute/zone us-central1-f
```


---

## ☸️ Create Kubernetes Cluster

### 3. Create a GKE Cluster with Autoscaling

Create a new GKE cluster. This command sets up a cluster with autoscaling enabled, specifying the minimum and maximum number of nodes.

```bash
CLUSTER_NAME=lab1-cluster

gcloud beta container clusters create $CLUSTER_NAME \
  --cluster-version=latest \
  --machine-type=n1-standard-4 \
  --enable-autoscaling \
  --min-nodes=1 \
  --max-nodes=3 \
  --num-nodes=1
```

### 4. Get Cluster Credentials and List Nodes

Once the cluster is created, get the credentials to interact with it using `kubectl` and list the nodes to verify the setup.

```bash
gcloud container clusters get-credentials $CLUSTER_NAME
kubectl get nodes
```


---

## 📦 Deploy TensorFlow Model to GKE

### 5. Apply ConfigMap for Model Path

The ConfigMap tells TensorFlow Serving where to find the model.

```bash
kubectl apply -f tf-serving/configmap.yaml
```

### 6. Deploy TensorFlow Serving

Deploy the TensorFlow Serving image as a deployment in your GKE cluster.

```bash
kubectl apply -f tf-serving/deployment.yaml
```

### 7. Expose the Deployment as a LoadBalancer Service

Create a service of type LoadBalancer to expose the TensorFlow Serving deployment to the internet.

```bash
kubectl apply -f tf-serving/service.yaml
```

### 8. Get External IP Address

Wait for a few minutes for the LoadBalancer to provision an external IP address for your service.

```bash
kubectl get svc image-classifier
```

Look for the `EXTERNAL-IP` value in the output. You might need to run this command a few times until the IP is assigned.


---

## ✅ Test Model Deployment

### 9. Send a Prediction Request

Once you have the `EXTERNAL_IP`, send a sample prediction request using `curl`. Make sure to replace `[EXTERNAL_IP]` with the actual IP address you obtained in the previous step.

```bash
curl -d @locust/request-body.json -X POST http://[EXTERNAL_IP]:8501/v1/models/image_classifier:predict
```


---

## 📈 Set Up Horizontal Pod Autoscaler (HPA)

### 10. Create Horizontal Pod Autoscaler

Configure HPA to automatically scale your `image-classifier` deployment based on CPU utilization. The target CPU utilization is 60%, and the number of replicas will scale between 1 and 4.

```bash
kubectl autoscale deployment image-classifier --cpu-percent=60 --min=1 --max=4
```

### 11. Check HPA Status

Verify the HPA has been created and see its current status.

```bash
kubectl get hpa
```

---

## 🧪 Load Test with Locust

### 12. Run Locust Load Test

Navigate to the `locust` directory and run the load test in headless mode. Replace `[EXTERNAL_IP]` with your service's external IP. This command will gradually increase the load.

```bash
cd locust
locust -f tasks.py \
  --headless \
  --users 32 \
  --spawn-rate 1 \
  --step-load \
  --step-users 1 \
  --step-time 30s \
  --host http://[EXTERNAL_IP]:8501
```


---

## 🎛️ GKE Monitoring 

- **Workloads View**: Navigate to your GKE cluster in the GCP Console, then go to "Workloads" to see your `image-classifier` deployment and the number of pods.

- **Node Pools View**: Check the "Node Pools" section for your cluster to see if new nodes were provisioned to handle the increased load.
![Screenshot 2025-05-07 203637](https://github.com/user-attachments/assets/227fd839-083d-4058-9355-5092c1e90078)



---



---

## 📚 References

- [TensorFlow Serving Documentation](https://www.tensorflow.org/tfx/guide/serving)
- [Google Kubernetes Engine (GKE) Documentation](https://cloud.google.com/kubernetes-engine/docs)
- [Google Cloud Labs](https://www.cloudskillsboost.google/focuses/17649?parent=catalog)
- [Horizontal Pod Autoscaler (HPA)](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)
- [Locust Load Testing](https://locust.io/)

---


