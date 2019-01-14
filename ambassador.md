1. If your using a modern k8s you'll need kubectl apply -f https://www.getambassador.io/yaml/ambassador/ambassador-rbac.yaml
2. kubectl apply -f ambassador-svc.yaml
3. kubectl apply -f cockroach-single.yaml
4. kubectl exec -it cockroachdb-0 bash - apt-get update && apt-get install -y curl
5. kubectl exec cockroachdb-0 /cockroach/cockroach -- user set rcjgo --insecure && \
  kubectl exec cockroachdb-0 /cockroach/cockroach -- sql --insecure -e 'CREATE DATABASE rcj' && \
  kubectl exec cockroachdb-0 /cockroach/cockroach -- sql --insecure -e 'GRANT ALL ON DATABASE rcj TO rcjgo'
6. curl -O https://raw.githubusercontent.com/davefinster/rcj-go/master/tables.sql
7. ./cockroach sql --user=rcjgo --database=rcj --insecure < tables.sql
8. kubectl create secret generic google-sheets-credential --from-file=./path/to/credential.json
9. kubectl apply -f rcj-go.yaml
10. kubectl apply -f rcj-go-mapping.yaml
11. kubectl get --namespace=cert-manager pods
