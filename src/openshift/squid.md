# Running Squid in a container

I had been running Squid on a physical machine before, but I wanted to move it into a container to make it easier to manage and deploy.

I used to run Squid on some old leftover hardware, then I started thinking: why not just run it as a container or pod instead? In my case, I have an OpenShift cluster but you can run it on vanilla Kubernetes or directly in Docker or Podman if you wanted to.

## What is in the project

The project is split into a few parts:

- `squid.conf` for the main Squid configuration
- `mime.conf` for MIME type mappings
- `configmap.yaml` to mount the config into the container
- `deploy.yaml` for the Squid deployment
- `svc.yaml` to expose the proxy on port `3128`
- `ingress.yaml` for HTTP ingress
- `squid-scc.yaml` for OpenShift security context constraints
- `README` with basic Docker build and run commands

## Building the container

The project README shows a simple Docker build and push flow:

```bash
docker build -t squid-proxy:v1.0 --omit-history .
docker push squid-proxy:v1.0
```

The README also shows how the image was tested interactively:

```bash
docker run -it localhost/squid-proxy:v1.0 /bin/bash
```

The actual `Dockerfile` or `Containerfile` is not in this folder, so this page is mostly about how the proxy is configured and deployed rather than how the image is built internally.

## Squid config

There are two places where the Squid config shows up in this project:

- a standalone `squid.conf`
- an embedded version inside `configmap.yaml`

The general pattern is the same in both places: Squid listens on `3128`, defines ACLs, and uses local disk for cache and runtime data.

For example:

```conf
http_port 3128
cache_dir ufs /var/spool/squid 100 16 256
```

A few things that stand out in the config:

- local source networks are defined with `acl localnet`
- safe ports and `CONNECT` ACLs are defined
- local domains and specific destination networks are allowed directly
- all other traffic can be forwarded through an upstream proxy with `cache_peer`

The config also references writable paths like:

- `/var/spool/squid`
- `/var/log/squid`
- `/var/run/squid`

From the manifests, I can clearly see storage mounted for `/var/spool/squid`. The log and runtime paths are in the config too, but how those are prepared inside the container depends on the image and entrypoint, and those files are not part of this folder.

## How the config gets into the container

The Kubernetes setup uses a ConfigMap called `squid-config` and mounts `squid.conf` into the container:

```yaml
volumes:
  - name: squid-config
    configMap:
      name: squid-config
```

and:

```yaml
volumeMounts:
  - name: squid-config
    mountPath: /etc/squid/squid.conf
    subPath: squid.conf
```

There is also a second ConfigMap for `mime.conf`.

That makes it easier to update the proxy config without rebuilding the image.

## Running the container

The README shows the image being tested locally with:

```bash
docker run -it localhost/squid-proxy:v1.0 /bin/bash
```

That part is the same no matter where you run it. If the image works, it can run with Docker, Podman, Kubernetes, OpenShift, or any other platform that supports containers.

## Deploying it on Kubernetes or OpenShift

The deployment runs a single Squid replica and starts it through an entrypoint script:

```yaml
containers:
  - name: squid
    image: squid-proxy:v1.0
    command: ["/bin/bash", "-c", "/usr/local/bin/entrypoint.sh"]
```

The container exposes port `3128`:

```yaml
ports:
  - containerPort: 3128
    protocol: TCP
```

The deployment also mounts an `emptyDir` volume to `/var/spool/squid`, which is where Squid keeps cache data in this setup.

## Service exposure

The Service exposes Squid as a `LoadBalancer` on port `3128`:

```yaml
spec:
  type: LoadBalancer
  ports:
    - port: 3128
      targetPort: 3128
```

There is also an ingress manifest in the project, but for a forward proxy the `LoadBalancer` Service is the part that matters most.

## OpenShift-specific setup

There is an SCC manifest called `squid-scc.yaml` for OpenShift. It allows the container to run with the security settings this image expects.

From the deployment:

- a dedicated service account is used
- `fsGroup: 999` is set
- `allowPrivilegeEscalation: true` is enabled in the container security context

I do not see the ServiceAccount manifest itself in this folder, but the deployment clearly expects one named `squid-proxy`.

From that, it looks like the deployment was adjusted so Squid would run properly inside OpenShift with writable mounted paths and a custom runtime user setup.

## PVCs vs current deployment

The project also includes `pvc.yaml` for cache and logs, but the current deployment manifest uses `emptyDir` instead of the PVCs.

So at the moment:

- cache is ephemeral
- the pod can be recreated without keeping cached data

If persistent cache or logs are important, the deployment would need to mount those PVCs instead of using `emptyDir`.

## What this setup looks like in practice

In practice, the setup is basically this:

1. Build and push a Squid container image
2. Put `squid.conf` and `mime.conf` in ConfigMaps
3. Deploy the container with writable mounted storage
4. Expose it with a Service on port `3128`
5. Add the OpenShift-specific security settings if running on OpenShift

The part I like most with this setup is that the proxy config is kept outside the container image. That makes it easy to change ACLs, upstream proxy settings, and local domain rules without rebuilding the image. In this case the files are mounted with `subPath`, so the pod still needs a restart before Squid picks up the changes.
