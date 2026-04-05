# n8n en OKD con Helm (oficial)

Este directorio despliega n8n con el chart oficial mantenido en Artifact Hub (`8gears/n8n`) usando Argo CD.

## Recursos

- `namespace.yml`: namespace `n8n`.
- `certificate.yml`: certificado TLS de `n8n-pro.apps.okd.pro.perelohome.com` usando `cloudflare-clusterissuer`.
- `application.yml`: `Application` de Argo CD que instala el chart Helm OCI y configura:
  - Ingress TLS con `n8n-server-tls-secret`
  - URL base/webhook en HTTPS
  - zona horaria `Europe/Madrid`
  - persistencia `10Gi` en `rook-ceph-block`

## Nota importante

Si quieres usar otro host o `secretName`, cambia ambos en:

- `certificate.yml` (`spec.secretName` y `spec.dnsNames`)
- `application.yml` (`ingress.hosts`, `ingress.tls.secretName`, y `main.config.n8n.*url*`)
