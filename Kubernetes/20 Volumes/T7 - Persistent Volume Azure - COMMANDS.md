1. A list of available StorageClass

  ```
  kubectl get sc
  ```

2. StorageClass details 

  ```
  kubectl describe sc default
  kubectl describe sc managed-premium
  ```

3. StorageClass for Azure Files  
  
  ```yaml
  kind: StorageClass
  apiVersion: storage.k8s.io/v1
  metadata:
    name: azurefile
  provisioner: kubernetes.io/azure-file
  mountOptions:
    - dir_mode=0777
    - file_mode=0777
    - uid=1000
    - gid=1000
    - mfsymlinks
    - nobrl
    - cache=none
  parameters:
    skuName: Standard_LRS
  ```
  
4. Create `PersistanceVolumeClaim` with Azure Files  

  ```yaml
  apiVersion: v1
  kind: PersistentVolumeClaim
  metadata:
    name: azurefile
  spec:
    accessModes:
      - ReadWriteMany
    storageClassName: azurefile
    resources:
      requests:
        storage: 10Gi
  ```

5. Create `Deployment` with Azure Files Volume   

  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: netcoresample
    labels:
      app: sample
  spec:
    replicas: 2
    selector:
      matchLabels:
        app: sample
    template:
      metadata:
        labels:
          app: sample
      spec:
        containers:
          - name: netcoresample
            image: mcr.microsoft.com/dotnet/core/samples:aspnetapp
            ports:
              - containerPort: 80
            volumeMounts:
              - mountPath: "/mnt/test"
                name: volume
        volumes:
          - name: volume
            persistentVolumeClaim:
              claimName: azurefile
  ```

6. Create new file in Azure Files (using Azure Portal). Use `kubectl exec` on any container and list files inside `/mnt/test`.

  ```shell
  kubectl exec <pod_name> -- sh -c "ls /mnt/test"
  ```

  ```shell
  kubectl exec <pod_name> -- sh -c "cat /mnt/test/<file_name>"
  ```
