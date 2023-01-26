---
layout: posts
share: true
title:  "Injecting Config as Environment Variables from Hasicorp's Consul & Vault"
date:   2021-11-10 20:46:34 +0100
categories: Kubernetes, Neo4j, Solution Architecture, Hasicorp, Vault, Consul
---

Continuing with the theme of Kubernetes, I have recently built out a solution to inject environment variables into containerised applications from Hasicorp Consult and Vault Key Value (KV) engine, which might be considered as a first step in realising Hashicorp's Service Mesh. Installing both Consul and Vault via helm with the KV Engine is fairly straightforward. Supplying these KV's as environment variables to the containerised applications in Kubernetes, however, requires a bit more thought. Two different approaches are required to lift in values from Consul and Vault which makes things even more interesting. The approach I took was to write the KV's to file before they are exposed as ENVs in the container, which is less than ideal. As a side note, it might be cleaner and simpler to manage config at the application layer by calling Consul and Vault's HTTP API. That is another approach which I'm not going to talk about here. 

For the purposes of this explanation I'm supplying environment variables to an application called Hasura graphQL API. Config is held in Consul and Vault in their respective KV Engines. 

![GitHub Logo](/assets/images/containers.jpg)
Source: https://www.pexels.com/

## Fetching KV's from Consul using envconsul 

Hasicorp provide a application called envconsul to fetch KV's from consul. I used that to write the KV's to a mounted volume ready to run as a bash script to expose them in the main application container. Note the use of the sed command to substitue the export statement. This achieves a file with lines that read `export key value` which can be run as a bash script to expose those KV's as ENVs in the main application container.

```
initContainers:
- name: envconsul
        image: hashicorp/envconsul:alpine
        {{ if eq .Values.application.namespace "hasura" }}
        command: ['sh','-c','envconsul -consul-addr=consul-consul-server.consul.svc.cluster.local:8500 -pristine -prefix staging/hasura env | sed "s/.*/export &/" > /consul/config/staging-hasura']
- mountPath: /consul/config/
        name: consul
```

## Fetching KV's from Vault using Vault Annotations

To fetch secrets from Vault I used the annotations shown in the snippet below which are patched to the deployment for Hasura. Note the `vault.hashicorp.com/agent-inject: "true"` label in the code snippet below which activates the Envoy sidecar although there is more going on here including an initContainer for Vault to initiate the mTLS mechanism so I'd recommend reading the docs. See https://www.vaultproject.io/docs/platform/k8s/injector/annotations. 

The label I wanted to give special mention to is `vault.hashicorp.com/agent-inject-template-staging-hasura:` which utilises a GO template. The template shown in the code snippet below writes the key value pairs in vault (prefixed with the word export) to file, ready to run as a bash script. This writes the same file output as the command for envconsul above. 

```
`kubectl patch deployment hasura --patch "$(cat staging-vault-patch.yaml)" -n hasura`
```

```
spec:
  template:
    metadata:
      annotations:
        vault.hashicorp.com/agent-inject: "true"
        vault.hashicorp.com/agent-inject-status: "update"
        vault.hashicorp.com/agent-inject-secret-staging-hasura: "staging/hasura"
        vault.hashicorp.com/agent-inject-template-staging-hasura: |
          {{ with secret "staging/hasura" -}}
            {{ range $k, $v := .Data }}
              export {{ $k }}={{ $v }}
            {{ end }}
          {{- end }}
        vault.hashicorp.com/role: "vault-hasura-service-account"
        vault.hashicorp.com/log-level: "debug"
```

Note that Vault also requires you to set up a Kubernetes Service Account `serviceAccountName: vault-service`. Refer to the Hashicorp docs for more information on this.

## Exposing the KV's into the main application container

Hasura deployment spec can be updated with a command statement: `command: ['sh', '-c', '. vault/secrets/staging-hasura && . consul/config/staging-hasura && graphql-engine serve']`. Using sh with the dot command to execute the bash scripts exposing the KV's as environment variables in the main application container, and then launching into the application itself. If you take this approach, be aware that the Kubernetes command statement will take precedence over any entrypoint command used in the dockerfile. If you are utilising a third party image that utilises an entrypoint statement you may need to create a custom dockerfile to make this approach work.

