# Kubernetes TLS Config

This is my post install on a fresh k3s kubernetes cluster on [Hetzner](https://www.hetzner.com) cloud.  
I used the excellent [Kube-Hetzner](https://github.com/kube-hetzner/terraform-hcloud-kube-hetzner) terraform setup for
Hetzner Cloud and performed the following post config to setup TLS and deploy a secure WHOAMI sample application.  

---

## Issuing Certificates

There are four main alternatives to issue certificates on kubernetes with `Traefik` Proxy.
1. **Cloud provider's load balancer** - may have support to manage certificates.  
   This option is not standard, will vary between cloud providers with proprietary APIs.

2. **Traefik** `--certificatesresolvers` declared in Traefik `additionalArguments:` 
   configuration section.  
   This a popular approach where:
     - Certificates resolvers are declared only once per the cluster.
     - Can be requested by an `Ingress` resource in the `annotations:` section refering to
       `router.tls.certresolver` and certificate definitions in `tls.hosts` and `tls.secretName`
       entries.
     - Can be requested by an `IngressRoute` resource without annotations. Everything goes in
       the `tls` section, in `tls.certResolver`, `tls.domains` and `tls.options` attributes.

   The downside:
     - Won't work with multiple instances of Traefik (HA) as there is no way to ensure the 
       correct instance receives the challenge request and subsequent responses.
     - Certificate resolvers configurations are mixed with the static configuration of Traefik 
       Proxy.

3. **Cert-Manager with Ingress** to manage certificates automatically.
   This is another popular solution that:
     - Cert-Manager is not affected by Traefik HA.
     - Separates certificate management configuration from Traefik Proxy configuration.
     - Has full integration for `Ingress` resources since `Cert-Manager` "understands" `Ingress`
       resources. `Cert-Manager` will create temporary challenge response services being able to
       issue the certificate by the `Ingress` resource request.

   The downside:
     - `Cert-Mangager` does **not** integregate with `IngressRoute` resources. There is a gap here,
       `IngressRoute` integrates with `Traefik` `certificateresolvers` but not with `Cert-Manager` to auto 
       issue certificates. Certificates must be requested either by an `Ingress` or by a `Certificate`
       resource.

4. **Cert-Manager with manual first issue** that manages certificates renewal.  
   This is the approach I like best and is the one described in this document.
     - Cert-Manager is not affected by Traefik HA.
     - Separates certificate management configuration from Traefik Proxy configuration.
     - Compatible with `Ingress` resources without annotations (or optional annotation to use HTTPS only).
     - Compatible with `IngressRoute`.
     - Although certificate issue is manual it is therefore manually verified.
     - Although certificate issue is manual, renewal is automatic.

   The downside:
     - First issue of a certificate is manual (but easy).

As mentioned above I do prefer to user `Cert-Manager` and issue the certificate for HTTPS manually and
have the benefit of automatic renewal and use only `IngressRoute` CRD rosources. This document will walk
you through the configurations necessary to use this approach.

---

### #0. Before anything you need to have your domain pointing to your cluster external IP.

This documentation requires that your DNS is already configured.  
  
In the example we will also redirect www.domain URLs to their short domain equivalent. It is required
that www and root addresses are both mapped in your DNS configuration, and it is OK to use
the general *.domain CNAME for this.  
  
On the other hand, when requesting a certificate with http challenge solvers every domain should be
explicity defined (*.domain is only supported through dns challenge solvers).  

---

### Chapter #1 - Create Cluster Issuers

Issuers is your CA representation. In this document we will be issuing LetsEncrypt certificates (staging and
production), this the Issuer will be a service to interact with LetsEncrypt authority.  
  
Cluster Issuers only differ from regular Issuer in the fact that they have no namespace and will be shared
by IngressRoutes on all namespaces.

Instructions in [Chapter 01-cluster-issuer](01-cluster-issuer/README.md).

---

### Chapter #2 - whoami app

Install the kute [whoami](https://hub.docker.com/r/traefik/traefikee-webapp-demo) webserver from Traefik dockerhub. This version may be configured to produce an ASCII
message for the console.  
  
This example installation goes step by step from issuing a letsencrypt staging 
certificate up to deploying the websecure application using Traefik CRD provider with production certificates and a set of middlewares for HTTP redirection, SSL strong security and 'www' prefix stripping.

Instructions in [Chapter #2 - whoami](02-whoami/README.md).

---

### References

This document is a compilation of best practices and instructions collected on the
internet. Here I name some of the main resources I used to create this repository.  
- APIs:
  - [Kubernetes API](https://kubernetes.io/docs/reference/kubernetes-api/)
  - [Traefik](https://doc.traefik.io/traefik/)
  - [cert-manager](https://cert-manager.io/docs/)
- Tutorials:
  - [Kube-Hetzner](https://github.com/kube-hetzner/terraform-hcloud-kube-hetzner) - Terraform template for Hetzner
  - [Workshop: Getting Started with Traefik](https://www.youtube.com/watch?v=CL5Cxxz-yHo&list=PL32htBncnWSjcoL3MwN-mT6rG1u3viRDI) - Traefik Labs tutorial with Jakub Hajek - Part #1
  - [Workshop: Advanced Load Balancing with Traefik 2.5](https://www.youtube.com/watch?v=eUlAS-FdELg&list=PL32htBncnWSjcoL3MwN-mT6rG1u3viRDI) - Traefik Labs tutorial with Jakub Hajek - Part #2
