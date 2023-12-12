# Ingress configuration
Installing JWTA is very easy., but maybe you need some help on configuring your Ingress. Following you will find some special actions you must perform on the different types of ingresses that JWTA supports in order to make them work with external auth.


## Ingress Nginx
TBD


## NGINX Ingress
You need to modify the deployment of the ingress to enable snippets (snippets are the way JWTA configures your ingress to support external authorization).

```yaml
  args:
    - -nginx-configmaps=$(POD_NAMESPACE)/nginx-config
    - **-enable-snippets**
```

## Traefik
You need to add some args to the deployment of the Traefik controller for: enable CRD's and enable ingress

```yaml
  args:
    - --api.insecure
    - **--providers.kubernetesingress**
    - --accesslog
    - **--providers.kubernetescrd**
    - **--providers.kubernetescrd.allowcrossnamespace**
    - **--providers.kubernetescrd.allowemptyservices**
    - **--providers.kubernetescrd.allowexternalnameservices**
    - --log.level=info
```

In addition you need to add some permisions to the cluster role. Following you can find a YAML for modifying the cluster role.

```yaml
ClusterRole
metadata:
  name: traefik-role
  - apiGroups:
      - extensions
      - networking.k8s.io
      - traefik.io
      - traefik.containo.us      
    resources:
      - ingresses
      - ingressclasses
      - middlewares
      - middlewaretcps
      - ingressroutes
      - traefikservices
      - ingressroutetcps
      - ingressrouteudps
      - tlsoptions
      - tlsstores
      - serverstransports
```

## Next step
Everyting is ready! Now you can create your first authorizator. Continue to a [basic JWT Authorizator deployment](/scenarios?id=basic-scenario-a-static-html-application).

