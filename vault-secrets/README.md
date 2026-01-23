# Configure vault and external-secrets on a kubernetes cluster

## Creamos en argocd la aplicacion 
Una vez creada la aplicacion, sincronizamos solamente vault-app.yml

## Configuramos vault:
Una vez levantado todo, aplicamos:
```bash
kubectl exec -ti vault-0 -n vault -- sh
vault operator init #Esto genera las passwords tanto del seal como el token de root MUY IMPORTANTE GUARDAR DE MANERA SEGURA!
```
Ahora hacemos unseal del vault:
```bash
vault operator unseal #aplicamos 3 veces con los passwords generados en el paso anterior
```
Hacemos login a vault:
```bash
vault login #ponemos la password root
vault secrets enable -path=secret kv-v2
vault kv put secret/demo user=admin password=superpass #Esto genera un password de prueba
```
Creamos la policy:
```bash
vault policy write external-secrets - <<EOF
path "secret/data/*" {
  capabilities = ["read"]
}
EOF
```
Generamos el token que se va a usar por external-secrets
```bash
vault token create -policy=external-secrets #GUARDAMOS EL TOKEN (SOLO DURA 760H)
```
Ahora creamos un secret en kubernets para el external-secrets: 
```bash
kubectl create secret generic vault-token \
  --from-literal=token=<EL_TOKEN_QUE_TE_DIO_VAULT> \
  -n external-secrets
```
## Sincronizamos ahora en argocd la otra aplicacion "external-secrets"
Arrancamos la aplicacion y esperamos que se creen todos los recursos

## Generamos el clusterSecretStore:
creamos el archivo clustersecretstore.yaml con el siguiente contenido:
```bash
apiVersion: external-secrets.io/v1beta1
kind: ClusterSecretStore
metadata:
  name: vault-backend
spec:
  provider:
    vault:
      server: "http://vault.vault:8200"
      path: "secret"
      version: "v2"
      auth:
        tokenSecretRef:
          name: vault-token
          key: token
          namespace: external-secrets
```
y aplicamos con
```bash
kubectl apply -f clustersecretstore.yaml
```

## Creamos un externalsecret de prueba:
Creamos el archivo demo-externalsecret.yaml:
```bash
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: demo-secret
  namespace: default
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: vault-backend
    kind: ClusterSecretStore
  target:
    name: demo-secret
    creationPolicy: Owner
  data:
    - secretKey: user
      remoteRef:
        key: demo
        property: user
    - secretKey: password
      remoteRef:
        key: demo
        property: password
```
y aplicamos:
```bash
kubectl apply -f demo-externalsecret.yaml
```

## Revisamos si se genera bien el secret:
```bash
kubectl get secret demo-secret -o yaml
kubectl get externalsecret -A
```

## Habilitamos el acceso web:
Sincronizamos en argocd el certificado y el ingress y metemos en nuestro DNS el acceso