# Deploy Mosquitto MQTT Broker on Kubernetes

## Deskripsi
Deploy Mosquitto MQTT broker menggunakan **Deployment + NodePort Service** di cluster Kubernetes. Konfigurasi broker dikelola dengan **ConfigMap** dan data disimpan persisten menggunakan **PersistentVolumeClaim**.

## Resource Kubernetes
- **ConfigMap**: `mosquitto-config` berisi `mosquitto.conf` (listener 1883, allow anonymous)  
- **PersistentVolume + PersistentVolumeClaim**: PV hostPath `/mosquitto/data` 1Gi, PVC `pvc-mosquitto-data` mount ke pod  
- **Deployment**: 1 replica, image `eclipse-mosquitto:2.0`, volume mount ConfigMap + PVC, resources: CPU 100m/500m, Memory 128Mi/512Mi  
- **Service (NodePort)**: `mosquitto-service`, port 1883, NodePort 31812, selector `app: mosquitto`  

## Create Manifest mosquitto-service.yaml
```
# --- ConfigMap ---
apiVersion: v1
kind: ConfigMap
metadata:
  name: mosquitto-config
  namespace: default
data:
  mosquitto.conf: |
    persistence true
    persistence_location /mosquitto/data/
    listener 1883
    allow_anonymous true

---

# --- Persistent Volume (hostPath) ---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-mosquitto-data
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  hostPath:
    path: /mosquitto/data
    type: DirectoryOrCreate

---

# --- Persistent Volume Claim ---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-mosquitto-data
  namespace: default
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: manual

---

# --- Deployment ---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mosquitto
  labels:
    app: mosquitto
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mosquitto
  template:
    metadata:
      labels:
        app: mosquitto
    spec:
      containers:
      - name: mosquitto
        image: eclipse-mosquitto:2.0
        ports:
        - containerPort: 1883
        volumeMounts:
        - name: mosquitto-config
          mountPath: /mosquitto/config
        - name: mosquitto-data
          mountPath: /mosquitto/data
        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "500m"
            memory: "512Mi"
      volumes:
      - name: mosquitto-config
        configMap:
          name: mosquitto-config
      - name: mosquitto-data
        persistentVolumeClaim:
          claimName: pvc-mosquitto-data

---
apiVersion: v1
kind: Service
metadata:
  name: mosquitto-service
  namespace: default
spec:
  selector:
    app: mosquitto
  type: NodePort
  ports:
    - name: mqtt
      protocol: TCP
      port: 1883
      targetPort: 1883
      nodePort: 31812

```


## Running Manifest mosquitto-service.yaml
```bash
kubectl apply -f mosquitto-deployment.yaml
```
```
kubectl get pods
kubectl get svc
```




## Cara Menggunakan
### Test koneksi MQTT broker
Pakai MQTT client di node
Install mosquitto-clients
```bash
sudo apt install mosquitto-clients -y
```

### NodePort
```bash
mosquitto_pub -h <NODE_IP> -p 31812 -t test -m "hello"
```
```bash
mosquitto_sub -h <NODE_IP> -p 31812 -t test
```