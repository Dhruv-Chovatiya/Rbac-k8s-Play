# Rbac-k8s-Play

Hi Checkers, please follow the below readme file for the better understanding.

# Generate .key for dev-user
openssl genrsa -out dev-user.key 2048

# Generate .csr for dev-user
openssl req -new \
    -key dev-user.key \
    -out dev-user.csr \
    -subj "/CN=dev-user/O=devteam"



# Generate .key for prod-user
openssl genrsa -out prod-user.key 2048

# Generate .csr for prod-user
openssl req -new \
    -key prod-user.key \
    -out prod-user.csr \
    -subj "/CN=prod-user/O=prodteam"


# Check that the files ca.crt and ca.key exist in the location.
ls ~/.minikube/

# generate .crt


# Generate .crt for dev-user
openssl x509 -req \
    -in dev-user.csr \
    -CA ~/.minikube/ca.crt \
    -CAkey ~/.minikube/ca.key \
    -CAcreateserial \
    -out dev-user.crt \
    -days 500


# Generate .crt for prod-user
openssl x509 -req \
    -in prod-user.csr \
    -CA ~/.minikube/ca.crt \
    -CAkey ~/.minikube/ca.key \
    -CAcreateserial \
    -out prod-user.crt \
    -days 500

# For dev-user:
kubectl config set-credentials dev-user \
    --client-certificate=dev-user.crt \
    --client-key=dev-user.key

kubectl config set-context dev-user-context \
    --cluster=minikube \
    --namespace=dev \
    --user=dev-user

# For prod-user:
kubectl config set-credentials prod-user \
    --client-certificate=prod-user.crt \
    --client-key=prod-user.key

kubectl config set-context prod-user-context \
    --cluster=minikube \
    --namespace=prod \
    --user=prod-user


kubectl create namespace dev
kubectl create namespace prod


kubectl config view

# Role & RoleBinding

kubectl apply -f dev-role.yaml
kubectl apply -f prod-role.yaml
kubectl apply -f dev-role-binding.yaml
kubectl apply -f prod-role-binding.yaml


kubectl config use-context dev-user-context

# Test accessing PROD namespace (should be forbidden)
kubectl get pods --namespace=prod  # Forbidden


kubectl config use-context prod-user-context

# Test accessing DEV namespace (should be forbidden)
kubectl get pods --namespace=dev  # Forbidden


kubectl config use-context minikube

# Test accessing DEV namespace (should succeed)
kubectl get pods --namespace=dev  # Success

# Test accessing PROD namespace (should succeed)
kubectl get pods --namespace=prod  # Success


