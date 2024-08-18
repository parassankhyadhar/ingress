# Ingress Controller Deployment

This repository contains the Kubernetes Ingress Controller manifest file and the Azure Pipeline YAML file to deploy the ingress controller to Azure Kubernetes Service (AKS). This setup is designed to route traffic to two different Spring Boot applications, "home-app" and "shopping-app," which are deployed on AKS.

## Repositories

- **[Home App](https://github.com/parassankhyadhar/home-app)**: Spring Boot application for the home service.
- **[Shopping App](https://github.com/parassankhyadhar/shopping-app)**: Spring Boot application for the shopping service.

## Ingress Controller Configuration

The Ingress Controller routes incoming HTTP requests to the appropriate backend service based on the URL path.

### Ingress Manifest File

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
    nginx.ingress.kubernetes.io/use-regex: "true"
    nginx.ingress.kubernetes.io/rewrite-target: /$2
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /home(/|$)(.*)
        pathType: Prefix
        backend:
          service:
            name: home-app-service
            port:
              number: 8080
      - path: /shopping(/|$)(.*)
        pathType: Prefix
        backend:
          service:
            name: shopping-app-service
            port:
              number: 8080
```
## Description

### Ingress Manifest File

- **Ingress Class**: Uses the `nginx` ingress controller.
- **Annotations**:
  - `ssl-redirect: "false"`: Disables SSL redirection.
  - `use-regex: "true"`: Allows the use of regex in path matching.
  - `rewrite-target: /$2`: Rewrites the target URL based on the regex capture group.
- **Rules**:
  - Routes traffic from `/home` to `home-app-service` on port 8080.
  - Routes traffic from `/shopping` to `shopping-app-service` on port 8080.

### Azure Pipeline Configuration

The Azure Pipeline YAML file automates the deployment of the Ingress Controller to your AKS cluster.

### Pipeline YAML File

```yaml
trigger:
- main

stages:
  - stage: Deploy_Ingress
    displayName: Deploy Ingress
    jobs:
      - job: Deploy
        displayName: Deploy Ingress
        pool:
          vmImage: 'ubuntu-latest'
        steps:
        - task: KubernetesManifest@1
          inputs:
            action: 'deploy'
            connectionType: 'kubernetesServiceConnection'
            kubernetesServiceConnection: 'akssc'
            manifests: '$(System.DefaultWorkingDirectory)/ingress.yaml'
```
### Description

- **Trigger**: The pipeline is triggered on changes to the `main` branch.
- **Stage**: The pipeline has a single stage called `Deploy_Ingress` to deploy the ingress.
- **Job**: The `Deploy` job uses the `KubernetesManifest` task to deploy the ingress manifest to AKS.
- **Service Connection**: The pipeline uses a Kubernetes service connection (`akssc`) to connect to your AKS cluster.

## Conclusion

This repository streamlines the deployment of an Ingress Controller on Azure Kubernetes Service (AKS), enabling efficient routing of traffic to your Spring Boot applications. By integrating the ingress manifest file and Azure Pipeline, you can automate the deployment process, ensuring that your applications are accessible through defined paths. 

For further customization or scaling, the provided configurations can be adapted to meet specific needs. Don't forget to review and adjust the ingress rules and pipeline settings based on your environment and application requirements.

For more information on the applications themselves, refer to the [Home App](https://github.com/parassankhyadhar/home-app) and [Shopping App](https://github.com/parassankhyadhar/shopping-app) repositories.
