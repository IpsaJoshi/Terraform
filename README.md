# Terraform
ssh-keygen -f my_key

scp -i assignment2-dev app.py Dockerfile Dockerfile_mysql mysql.sql requirements.txt 3.223.9.113:/home/ec2-user

docker build -t my_db -f Dockerfile_mysql . (maybe not necce)

ssh -i assignment2-dev 3.223.9.113   

Installing Docker: 

sudo yum update -y
sudo yum install docker -y                                                                              
sudo systemctl start docker
ls -ltra /var/run/docker.sock
sudo usermod -a -G docker ec2-user
docker ps

Installing kind:

docker rmi -f $(docker images -aq)
curl -sLo kind https://kind.sigs.k8s.io/dl/v0.11.0/kind-linux-amd64
sudo install -o root -g root -m 0755 kind /usr/local/bin/kind
rm -f ./kind

# Install kubectl – important! It should match cluster version 1.21
# https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html

curl -o kubectl https://s3.us-west-2.amazonaws.com/amazon-eks/1.21.2/2021-07-05/bin/linux/amd64/kubectl
chmod +x ./kubectl
mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$PATH:$HOME/bin
echo 'export PATH=$PATH:$HOME/bin' >> ~/.bashrc
kubectl version --short --client




cat > kind.yaml <<EOF
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  image: kindest/node:v1.19.11@sha256:07db187ae84b4b7de440a73886f008cf903fcf5764ba8106a9fd5243d6f32729
  extraPortMappings:
  - containerPort: 30000
    hostPort: 30000
  - containerPort: 30001
    hostPort: 30001
EOF
kind create cluster --config kind.yaml  
OR
kind create cluster --config kind.yaml --name assignment2

############## Inside EC2 #####################

aws configure
cd ~/.aws
vi crendentials

aws ecr get-login-password --region us-east-1 | docker login --username AWS 419810237977.dkr.ecr.us-east-1.amazonaws.com
sudo $(aws ecr get-login --no-include-email --region us-east-1)

sudo systemctl start docker


1.
kubectl cluster-info 
kubectl get pods --all-namespaces
kubectl get nodes

################################################################################

2.

k apply -f mysql-pod.yaml --namespace=sqldb
k apply -f webapp-pod.yaml --namespace=webapp

k get pods --namespace=sqldb
kubectl get pods -o wide --namespace=sqldb

k get pods --namespace=webapp
kubectl get pods -o wide --namespace=webapp   

k port-forward pod/webapp-pod --namespace=webapp 8888:8080
#new terminal
curl localhost:8888

K logs webapp-pod --namespace=webapp 

#################################### REPLICA ################################

k apply -f mysql-replicaset.yaml --namespace=sqldb
k get pods --namespace=sqldb
  
k apply -f webapp-replicaset.yaml --namespace=webapp
k get pods --namespace=webapp

################################### DEPLOY ######################################

3.

k apply -f mysql-deployment.yaml --namespace=sqldb
k get pods --namespace=sqldb
k get all -n sqldb

k apply -f webapp-deployment.yaml --namespace=webapp
k get pods --namespace=webapp
k get all -n webapp

################################### SERVICE #####################################

4.

k apply -f mysql-service.yaml --namespace=sqldb

k apply -f webapp-service.yaml --namespace=webapp

k get all -n sqldb
k get all -n webapp

k port-forward service/webapp-service --namespace=webapp 8080:8080

curl localhost:8080

browser

kubectl delete pods --all -n webapp
