## Scaling self-hosted runners with Kubernetes using ARC

```
az login
az group create --name AKSCluster -l westeurope
az aks create --resource-group AKSCluster \
--name AKSCluster \
--node-count 3 \
--enable-addons monitoring \
--generate-ssh-keys
az aks get-credentials --resource-group AKSCluster --name AKSCluster
```

Install Cert-Manager

```
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.14.5/cert-manager.yaml
```

1. Deploy ARC to your cluster. Make sure you change its version to an up-to-date version

   ```
   kubectl apply -f https://github.com/actions/actions-runner-controller/releases/download/v0.23.7/actions-runner-controller.yaml
   kubectl apply -f https://github.com/actions/actions-runner-controller/releases/download/gha-runner-scale-set-0.9.1/actions-runner-controller.yaml
   ```

2. Create a PAT in GitHub with the **repo** scope.
   
3. Now, save the token as a secret in Kubernetes:

   ```
   kubectl create secret generic controller-manager -n actions-runner-system --from-literal=github_token=<YOUR_TOKEN>
   ```

4. Create a file called **runnerdeployment.yml** in your repository with the following content. Replace the repository owner and name with the values for your repository:

   ```
   apiVersion: actions.summerwind.dev/v1alpha1
   kind: RunnerDeployment
   metadata:
     name: example-runnerdeploy
   spec:
     replicas: 1
     template:
       spec:
         repository: wulfland/GitHubActionsCookbook
   ```

5. Apply RunnerDeployment to your cluster.

   ```
   kubectl apply -f runnerdeployment.yml
   ```

6. Now, you should have one runner and two pods running:

   ```
   kubectl get runners
   kubectl get pods
   ```

7. GitHub Action file

   ```
   self-hosted.yml
   ```
