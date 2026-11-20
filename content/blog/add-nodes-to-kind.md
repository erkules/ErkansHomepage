---
title: Add Nodes to kind
date: 2026-06-08
tags:
  - kubernetes
categories:
  - Blog
---

Gegeben ein Single-Node kind Cluster:

```bash
$ kubectl get nodes
NAME                 STATUS   ROLES           AGE    VERSION
kind-control-plane   Ready    control-plane   3d7h   v1.34.0
```

Jetzt will ich den Cluster erweitern. Kind sieht das ja eigentlich nicht vor.
Ist aber eh nur kubeadm@Docker.

Also bereiten wir mal ein paar Worker vor.

```bash
for i in 1 2 3 ; do
docker container run --name worker-$i --privileged --network kind  -d kindest/node:v1.34.0
done
```

Ja wir müssen den Container privileged laufen lassen.
Kind kann mehrere Cluster starten. Jeder bekommt ein eigenes Docker-Network.
Ergibt sich aus dem Namen oder einfach mit `docker container inspect ..` checken.


Jetzt schnell 0815 kubeadm-Magie.

Join Token erstellen:

```bash
$ docker exec kind-control-plane kubeadm token create --print-join-command
kubeadm join kind-control-plane:6443 --token 4tra6y.2nprsx83rl0ac9g1 --discovery-token-ca-cert-hash sha256:9a274c69fcf9bb2e62a612f54676747c00848bb9dae6c32619be304f820e5d9f 
```

und auf den neuen Workern ausführen

```bash
for i in 1 2 3 ; do
docker exec worker-$i kubeadm join kind-control-plane:6443 --token 4tra6y.2nprsx83rl0ac9g1 --discovery-token-ca-cert-hash sha256:9a274c69fcf9bb2e62a612f54676747c00848bb9dae6c32619be304f820e5d9f --ignore-preflight-errors=all
done
```

