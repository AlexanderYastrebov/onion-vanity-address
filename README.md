# onion-vanity-address 🧅

This tool generates Tor Onion Service v3 [vanity address](https://community.torproject.org/onion-services/advanced/vanity-addresses/) with a specified prefix.

Compared to [similar tools](#similar-tools), it uses the [fastest search algorithm](#the-fastest-search-algorithm) 🚀

## Usage

Install the tool locally and run:
```console
$ go install github.com/AlexanderYastrebov/onion-vanity-address@latest

$ onion-vanity-address allium
Found allium... in 12s after 558986486 attempts (48529996 attempts/s)
---
hostname: alliumdye3it7ko4cuftoni4rlrupuobvio24ypz55qpzjzpvuetzhyd.onion
hs_ed25519_public_key: PT0gZWQyNTUxOXYxLXB1YmxpYzogdHlwZTAgPT0AAAAC1ooweCbRP6ncFQs3NRyK40fRwaodrmH572D8py+tCQ==
hs_ed25519_secret_key: PT0gZWQyNTUxOXYxLXNlY3JldDogdHlwZTAgPT0AAAAQEW4Rhot7oroPaETlAEG3GPAntvJ1agF2c7A2AXmBW3WqAH0oUZ1hySvvZl3hc9dSAIc49h1UuCPZacOWp4vQ
```

or use the Docker image:
```console
$ docker pull ghcr.io/alexanderyastrebov/onion-vanity-address:latest
$ docker run  ghcr.io/alexanderyastrebov/onion-vanity-address:latest allium
```

To configure hidden service keypair decode base64-encoded secret key into `hs_ed25519_secret_key` file,
remove `hs_ed25519_public_key` and `hostname` files and restart Tor service:
```console
$ echo PT0gZWQyNTUxOXYxLXNlY3JldDogdHlwZTAgPT0AAAAQEW4Rhot7oroPaETlAEG3GPAntvJ1agF2c7A2AXmBW3WqAH0oUZ1hySvvZl3hc9dSAIc49h1UuCPZacOWp4vQ | base64 -d > /var/lib/tor/hidden_service/hs_ed25519_secret_key

$ rm /var/lib/tor/hidden_service/hs_ed25519_public_key
$ rm /var/lib/tor/hidden_service/hostname
$ systemctl restart tor

$ cat /var/lib/tor/hidden_service/hostname
alliumdye3it7ko4cuftoni4rlrupuobvio24ypz55qpzjzpvuetzhyd.onion
```

## Performance

The tool checks ~45'000'000 keys per second on a laptop:
```console
$ onion-vanity-address --timeout 20s goodluckwiththisprefix
Stopped searching goodluckwiththisprefix... after 20s and 959763220 attempts (47985799 attempts/s)
```

which is ~2x faster than `mkp224o`:
```console
$ timeout 20 docker run ghcr.io/cathugger/mkp224o:master -s -y goodluckwiththisprefix
sorting filters... done.
filters:
        goodluckwiththisprefix
in total, 1 filter
using 8 threads
>calc/sec:18497645.320881, succ/sec:0.000000, rest/sec:79.315507, elapsed:0.100863sec
>calc/sec:18884429.043617, succ/sec:0.000000, rest/sec:0.000000, elapsed:10.108983sec
```

In practice, it finds a 6-character prefix within a minute.
Each additional character increases search time by a factor of 32.

## Kubernetes

Run distributed vanity address search in Kubernetes cluster using the [demo-k8s.yaml](demo-k8s.yaml) manifest
without exposing the secret key to the cluster:

```console
$ # Locally generate secure starting key pair (or use existing one created by Tor)
$ onion-vanity-address start
Found start... in 1s after 26921387 attempts (43429741 attempts/s)
---
hostname: startxxytwan7gfm6ojs6d2auwhwjhysjz3c5j2hd7grlokzmd4reoqd.onion
hs_ed25519_public_key: PT0gZWQyNTUxOXYxLXB1YmxpYzogdHlwZTAgPT0AAACUwRne+J2A35is85MvD0Clj2SfEk52LqdHH80VuVlg+Q==
hs_ed25519_secret_key: PT0gZWQyNTUxOXYxLXNlY3JldDogdHlwZTAgPT0AAABgZ5a7kuS0N1jaA12gtsqI87RPS1eqSj4KWpwXukWtV7pFj6gS200J96P8JDWTpvx000KF3r4l+xYcIJszhPZk

$ # Edit demo-k8s.yaml to configure prefix, starting **public key**, parallelism, and resource limits 💸

$ # Create search job
$ kubectl apply -f demo-k8s.yaml
job.batch/ova created

$ # Check the job
$ kubectl get job ova
NAME   STATUS    COMPLETIONS   DURATION   AGE
ova    Running   0/999999      5s         5s

$ # Check pods
$ kubectl get pods --selector=batch.kubernetes.io/job-name=ova
NAME          READY   STATUS    RESTARTS   AGE
ova-0-tz27j   1/1     Running   0          7s
ova-1-zwlhl   1/1     Running   0          7s
ova-2-khl7f   1/1     Running   0          7s
ova-3-9l4z5   1/1     Running   0          7s
ova-4-tbx2m   1/1     Running   0          7s
ova-5-mpsz8   1/1     Running   0          7s
ova-6-xg7ft   1/1     Running   0          7s
ova-7-6zcn8   1/1     Running   0          7s
ova-8-cqrtj   1/1     Running   0          7s
ova-9-dtqhc   1/1     Running   0          7s

$ # Check resource usage
$ kubectl top pods --selector=batch.kubernetes.io/job-name=ova

$ # Wait for the job to complete
$ kubectl wait --for=condition=complete job/ova --timeout=1h
job.batch/ova condition met

$ # Job is complete
$ kubectl get job ova
NAME   STATUS     COMPLETIONS   DURATION   AGE
ova    Complete   1/999999      23m14s     23m44s

$ # Get found offset from the logs
$ kubectl logs jobs/ova
Found lukovitsa... in 23m14s after 1003371311076 attempts (719798516 attempts/s)
---
hostname: lukovitsa6jy7sldxvdw7wwzdmf5sezbwgr5uf57kkhi3jep25g2d2id.onion
offset: sgowAsMLwBk=

$ # Locally generate vanity key pair by offsetting the starting secret key
$ echo PT0gZWQyNTUxOXYxLXNlY3JldDogdHlwZTAgPT0AAABgZ5a7kuS0N1jaA12gtsqI87RPS1eqSj4KWpwXukWtV7pFj6gS200J96P8JDWTpvx000KF3r4l+xYcIJszhPZk | onion-vanity-address --offset=sgowAsMLwBk=
---
hostname: lukovitsa6jy7sldxvdw7wwzdmf5sezbwgr5uf57kkhi3jep27gzjlid.onion
hs_ed25519_public_key: PT0gZWQyNTUxOXYxLXB1YmxpYzogdHlwZTAgPT0AAABdFOqicgeTj8ljvUdv2tkbC9kTIbGj2he/Uo6NpI/XzQ==
hs_ed25519_secret_key: PT0gZWQyNTUxOXYxLXNlY3JldDogdHlwZTAgPT0AAAAoaPTTqGQGyF3aA12gtsqI87RPS1eqSj4KWpwXukWtVyHuiixSBYjSDLiBwGmeqebH1FX7vsHRPBrojpTFiCGQ

$ # Delete the job
$ kubectl delete job ova
job.batch "ova" deleted
```

## Similar tools

* [mkp224o](https://github.com/cathugger/mkp224o)
* [oniongen-go](https://github.com/rdkr/oniongen-go)

## The fastest search algorithm

Tor Onion Service [address](https://github.com/torproject/torspec/blob/main/rend-spec-v3.txt) is derived from ed25519 public key.
The tool generates candidate public keys until it finds one that has a specified prefix when encoded as onion address.

ed25519 keypair consists of:
* 32-byte secret key (scalar) - a random value that serves as the secret
* 32-byte public key (point) - derived by scalar multiplication of the base point by the scalar

ed25519 public key is 32-byte y-coordinate of a point on a [Twisted Edwards curve](https://datatracker.ietf.org/doc/html/rfc8032) equivalent to [Curve25519](https://datatracker.ietf.org/doc/html/rfc7748#section-4.1).

Both `mkp224o` and `onion-vanity-address` leverage additive properties of elliptic curves to avoid full scalar multiplication for each candidate key.
Addition of points requires expensive field inversion operation and both tools utilize batch field inversion (Montgomery trick)
to perform single field inversion per batch of candidate points.

The key performance difference is that while `mkp224o` uses point arithmetic that calculates both coordinates for each candidate point,
`onion-vanity-address` uses curve coordinate symmetry and calculates only y-coordinates to reduce number of field operations.

The algorithm has amortized cost **5M + 2A** per candidate key, where M is field multiplication and A is field addition.

See also:
* [vanity25519](https://github.com/AlexanderYastrebov/vanity25519) — Efficient Curve25519 vanity key generator.
* [wireguard-vanity-key](https://github.com/AlexanderYastrebov/wireguard-vanity-key) — Fast WireGuard vanity key generator.
* [age-vanity-keygen](https://github.com/AlexanderYastrebov/age-vanity-keygen) — Fast vanity age X25519 identity generator.
