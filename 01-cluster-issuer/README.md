## Chapter #1 - Create Cluster Issuers

An `Issuer` is your CA representation. In this document we will be issuing LetsEncrypt certificates
(staging and production), this `Issuer` will be a service to interact with LetsEncrypt authority.  
  
A `ClusterIssuer` only differ from regular `Issuer` in the fact that the first has no namespace and
are shared by `IngressRoutes` on all namespaces.

---

### #0. Pre Requisites
This documentation requires that your DNS is already configured.  
  
In the next chapter (`02-whoami`) we will set up www-url prefix stripping and it will be required that
both www and root addresses are mapped in your DNS configuration. It is OK to use the wildcard *.domain
CNAME for this.  
  
On the other hand, when issuing certificates with http challenge solvers every domain should be explicity
defined (wildcard domains are only supported through dns challenge solvers).  

---

### #1. Create certificate issuers

We will install cluster certificate issuers for both staging and production certificates.  
At this point no certificates are issued, the issuers will be ready to issue the first certificate, renewal
will be automatic.  

Create staging and production certificate `ClusterIssuers`:

```
kubectl apply -f 01-letsencrypt.yaml
```
  
Check they were created (cluster issuers have no namespace):

```
kubectl get clusterissuers

NAME                  READY   AGE
letsencrypt           True    2m
letsencrypt-staging   True    2m
```

Check that each one has a corresponding private-key secret in cert-manager namespace:
```
kubectl -n cert-manager get secret

NAME                                 TYPE      DATA   AGE
letsencrypt-private-key              Opaque    1      4m
letsencrypt-staging-private-key      Opaque    1      4m
```

### #Next

You can now proceed to the next chapter whre we will issue certificates for SSL connections and more:
[Chapter #2 - whoami](../02-whoami/README.md).
