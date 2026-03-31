# Installation:

on okd, you have to apply scc permisions. To install it execute the following:

```bash
oc adm policy add-scc-to-group anyuid system:serviceaccounts:data-analys
oc adm policy add-scc-to-group privileged system:serviceaccounts:data-analys

# confirm SA is using sts (if no response, is using default)
oc -n data-analys get sts airbyte-db -o jsonpath='{.spec.template.spec.serviceAccountName}{"\n"}'

# in case of default
oc auth can-i use scc/privileged --as=system:serviceaccount:data-analys:default -n data-analys

# force recreate
oc -n data-analys delete pod airbyte-db-0 --ignore-not-found
