# AWX-on-Kubernetes
AWX op Kubernetes met kind - Installatiehandleiding
AWX op Kubernetes met kind - Installatiehandleiding

Deze handleiding beschrijft een succesvolle installatie van AWX via de AWX Operator op een lokale Kubernetes-omgeving met behulp van kind.

📋 Vereisten

Ubuntu 22.04 (getest op een VM)

Internettoegang

Gebruiker met sudo-rechten

🔧 Stap 1: Vooraf benodigde tools installeren

sudo apt update && sudo apt install -y \
  make git curl wget unzip \
  htop net-tools ncdu bash-completion vim \
  apt-transport-https ca-certificates gnupg \
  docker.io containerd jq

sudo systemctl enable --now docker
sudo usermod -aG docker $USER
newgrp docker

📦 Stap 2: Installeer kubectl en kind

curl -LO https://dl.k8s.io/release/v1.33.1/bin/linux/amd64/kubectl
chmod +x kubectl
sudo mv kubectl /usr/local/bin/

curl -Lo ./kind https://kind.sigs.k8s.io/dl/latest/kind-linux-amd64
chmod +x kind
sudo mv kind /usr/local/bin/

☸️ Stap 3: Maak een Kubernetes cluster met kind

kind create cluster --name awx --image kindest/node:v1.28.0

Controleer of de cluster draait:

kubectl get nodes

🧰 Stap 4: Installeer de AWX Operator

git clone https://github.com/ansible/awx-operator.git
cd awx-operator
git checkout tags/2.19.1
make deploy

🧾 Stap 5: Maak je AWX instance

Maak een bestand ~/awx-instance.yaml aan met de volgende inhoud:

apiVersion: awx.ansible.com/v1beta1
kind: AWX
metadata:
  name: awx
  namespace: awx
spec:
  service_type: NodePort
  hostname: awx.local
  projects_persistence: true
  projects_storage_access_mode: ReadWriteOnce
  projects_storage_size: 5Gi

Pas toe:

kubectl apply -f ~/awx-instance.yaml

Controleer de pods:

kubectl get pods -n awx -w

🔐 Stap 6: Verkrijg het admin-wachtwoord

kubectl get secret awx-admin-password -n awx -o jsonpath="{.data.password}" | base64 -d && echo

🌐 Stap 7: Toegang tot de AWX webinterface

Start port-forwarding:

kubectl port-forward svc/awx-service -n awx 8080:80

Open in je browser:

http://localhost:8080

Login met:

Gebruikersnaam: admin

Wachtwoord: (zie stap 6)

✅ Status controleren

kubectl get pods -n awx
kubectl get svc -n awx
