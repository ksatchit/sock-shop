# Monitor Chaos on Sock-Shop

Chaos experiments on sock-shop app with grafana dashboard to monitor it. 

## Step-0: Obtain the demo artefacts

- Clone the sock-shop repo

  ```
  git clone https://github.com/ksatchit/sock-shop.git
  cd sock-shop
  ```


## Step-1: Setup Sock-Shop Microservices Application

- Create sock-shop namespace on the cluster

  ```
  kubectl create ns sock-shop
  ```

- Apply the sock-shop microservices manifests

  ```
  kubectl apply -f deploy/sock-shop/
  ```

- Wait until all services are up. Verify via `kubectl get pods -n sock-shop`

## Step-2: Setup the LitmusChaos Infrastructure

- Install the litmus chaos operator and CRDs 

  ```
  kubectl apply -f https://litmuschaos.github.io/pages/litmus-operator-v1.5.0.yaml
  ```

- Install the litmus-admin serviceaccount for centralized/admin-mode of chaos execution

  ```
  kubectl apply -f https://raw.githubusercontent.com/litmuschaos/pages/master/docs/litmus-admin-rbac.yaml
  ```

- Install the chaos experiments in admin(litmus) namespace

  ```
  kubectl apply -f https://hub.litmuschaos.io/api/chaos/1.5.0?file=charts/generic/experiments.yaml -n litmus 
  ```

- Install the chaos experiment metrics exporter and chaos event exporter

  ```
  kubectl apply -f deploy/litmus-metrics/01-event-router-cm.yaml
  kubectl apply -f deploy/litmus-metrics/02-event-router.yaml
  kubectl apply -f deploy/litmus-metrics/03-chaos-exporter.yaml
  ```

## Step-2: Setup the Monitoring Infrastructure

- Apply the monitoring manifests in specified order

  ```
  kubectl apply deploy/monitoring/01-monitoring-ns.yaml
  kubectl apply deploy/monitoring/02-prometheus-rbac.yaml
  kubectl apply deploy/monitoring/03-prometheus-configmap.yaml
  kubectl apply deploy/monitoring/04-prometheus-alert-rules.yaml
  kubectl apply deploy/monitoring/05-prometheus-deployment.yaml
  kubectl apply deploy/monitoring/06-prometheus-svc.yaml
  kubectl apply deploy/monitoring/07-grafana-deployment.yaml
  kubectl apply deploy/monitoring/08-grafana-svc.yaml
  ```

- Access the grafana dashboard via the NodePort (or loadbalancer) service IP or via a port-forward operation on localhost

  ```
  kubectl get svc -n monitoring 
  ```

  Default username/password credentials: `admin/admin`

- Add the prometheus datasource for Grafana via the Grafana Settings menu


- Import the grafana dashboard "Sock-Shop Performance" provided [here](https://raw.githubusercontent.com/ksatchit/sock-shop/master/deploy/monitoring/10-grafana-dashboard.json)


## Step-3: Execute the Chaos Experiments


- For the sake of illustration, let us execute a CPU hog experiment on the `catalogue` microservice & a Memory Hog experiment on 
  the `orders` microservice in a staggered manner
 

  ```
  kubectl apply -f chaos/catalogue/catalogue-cpu-hog.yaml
  ```

  Wait for ~60s

  ```
  kubectl apply -f chaos/orders/orders-memory-hog.yaml
  ```

## Step-4: Visualize Chaos Impact

- Observe the impact of chaos injection through increased Latency & reduced QPS (queries per second) on the microservices 
  under test. 


