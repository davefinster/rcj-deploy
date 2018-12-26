# Deploying RCJ to Kube

- Ensure that ```kubectl get nodes``` or a similarly basic command works
- Install Helm/Tiller. Helm is the client side component while Tiller is the server-side
  - Details can be found here: https://github.com/helm/helm/blob/master/docs/install.md
  - On Ubuntu, you can follow these instructions
```
sudo snap install helm --classic
kubectl create serviceaccount tiller --namespace=kube-system
kubectl create clusterrolebinding tiller-admin --serviceaccount=kube-system:tiller --clusterrole=cluster-admin
helm init --service-account=tiller
```
- With Helm installed, running the following to install cert-manager ```helm install --name cert-manager --namespace kube-system stable/cert-manager```
  - You may encounter an error about a resource already existing. In that case, use these steps
```
helm delete --purge cert-manager
helm install --name cert-manager --namespace kube-system stable/cert-manager --set createCustomResource=false
helm upgrade --install --namespace kube-system cert-manager stable/cert-manager --set createCustomResource=true
```
- Create a Cert Issuer tied to Lets Encrypt with ```kubectl apply -f cert-issuer.yaml```
- Provision an nginx based ingress with ```kubectl apply -f ingress-nginx.yaml```
- Using ```kubectl get --namespace=ingress-nginx services```, obtain the external IP of the load balancer
- Update the DNS entry you want to use for RCJ to point to this IP address. You can check propogation using https://www.whatsmydns.net/
- Generate the SSL certificate using ```kubectl apply -f cert.yaml```
- In order to use Google Sheets, we need some credentials for the app to use. Visit https://developers.google.com/sheets/api/quickstart/go and use the Enable button. Follow the steps and download the credentials.json file.
- Install the credentials into the cluster ```kubectl create secret generic google-sheets-credential --from-file=credential.json```
- Deploy the database ```kubectl apply -f cockroach-single.yaml```
- Create the database user using ```kubectl exec cockroachdb-0 /cockroach/cockroach -- user set rcjgo --insecure```
- Create the database using ```kubectl exec cockroachdb-0 /cockroach/cockroach -- sql --insecure -e 'CREATE DATABASE rcj'```
- Assign permissions using ```kubectl exec cockroachdb-0 /cockroach/cockroach -- sql --insecure -e 'GRANT ALL ON DATABASE rcj TO rcjgo'```
- Get a terminal on the database container using ```kubectl exec -it cockroachdb-0 bash``` and run the following
```
apt-get update
apt-get install -y curl
curl -O https://raw.githubusercontent.com/davefinster/rcj-go/master/tables.sql
./cockroach sql --user=rcjgo --database=rcj --insecure < tables.sql
```
- Deploy the backend with ```kubectl apply -f rcj-go.yaml```
- Deploy the frontend with ```kubectl apply -f rcj-react.yaml```
- Deploy the ingress entry ```kubectl apply -f ingress.yaml```


