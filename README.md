Introduction:
  This is a Helm chart for project elmanazala-meatshop
  sources:
    frontend: https://github.com/BRHM1/elmal7ma.git
    backend: https://github.com/BRHM1/g-back.git
  images:
    frontend: borhom11/frontend:1.0.5
    backend: eladwy/g-back

Prerequisites:
  storageClass: https://github.com/rancher/local-path-provisioner

  nginx-ingress-controller: |-
     # Add the Helm repository
     helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
     helm repo update
    # Install (with NodePort for bare metal/VMs)
    helm upgrade --install ingress-nginx ingress-nginx/ingress-nginx \
    --namespace ingress-nginx \
    --create-namespace \
    --set controller.service.type=NodePort \
    --set controller.hostNetwork=true  # Only for bare metal/VMs

  /etc/hosts:
    If you're running a kubeadm cluster make sure to add the node-ip of the node that running the ingress controller pod
    and map it to frontend.com backend.com so you can access the application on http://frontend.com:ingress_controller_svc_nodePort
    Steps:
      1- Excute kubectl get pods -n ingress-nginx -o wide 
      2- Excute ssh node-name 
      3- Excute ip addr and lookup the ip of interface
      4- Add Node-IP frontend.com backend.com to /etc/hosts 
Values.yaml:
  frontend:
    deployment: 
      name: <deployment-name>
      labels:
        <labels-in-key-value-pair-format>
      replicas: <num-of-pods>
      image: borhom11/frontend:1.0.5
  services:
    name: <frontend-svc-name>
    port: <svc-port>
  cm:
    name: <frontend-config-name>
  backend:
    deployment:
      name: <backend-deployment-name>
      labels:
        <labels-in-key-value-pair-format>
      replicas: <num-of-pods>
      image: eladwy/g-back
      secretName: <backend-secret-name>
    services:
      nodePort:
        name: <backend-nodeport-svc-name>
        port: <backend-nodeport-svc-port> 
      clusterIP:
        name: <backend-clusterip-svc-name>
        port: <backend-clusterip-svc-name>
    secret:
      name: <backend-secret-name>
      db_name: (e.g. meatshop) 
      port: (e.g. 3306) 
      user: (e.g. root) 
      password: <base64-encoded-password>
      host: <mysql-statefulset-pod>.<headless-service-name> e.g. mysql-0.mysql
      settings_module: bWVhdHNob3Auc2V0dGluZ3MuZGV2
      secret_key: Create one from https://djecrety.ir/ and paste it base64 encoded here..
      superuser_email: <super-user-email>
      superuser_username: <super-user-username>
      superuser_password: <super-user-password>
  mysql:
    statefulset:
      name: <statefulset-name>
      labels:
        <labels-in-key-value-pair-format>
      image: mysql
      storageClass: <name-of-storageClass-provisioned-on-cluster>
    secret:
      name: <db-secret-name>
      db: bWVhdHNob3A=
      root_password: <root-password-for-root-user>
      user: cm9vdA==
    cms:
      liveness:
        name: <liveness-probe-configMap-name>
      readiness:
        name: <readiness-probe-configMap-name>
    service:
      name: <mysql-headless-service-name>
    job:
      name: <mysql-init-job-name>

  ingress:
    name: <ingress-name>
    frontend_host: <frontend-host-in-/etc/hosts-file>
    backend_host: <backend-host-in-/etc/hosts-file>
    ingress_controller_svc_nodePort: <ingress-controller-service-nodePort-in-cluster>


Notes:
  - Any value encoded under secret leave it as it's
  - Any value under secret should be base64 encoded
  - Excute kubectl get svc -n ingress-nginx, to get the correct value of the ingress controller svc nodePort look at ingress-nginx-controller svc nodePort
  - Mysql image should be >= mysql:8.0.11 



# k8s-mal7ma
