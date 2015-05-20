# BOSH Release for root-env

Have you ever spent a lot of time inside BOSH VMs troubleshooting, and wish
you had your favorite editor configs, aliases, and other settings? Wish no more!

**root-env-boshrelease** Allows you to customize the root user environment inside
BOSH VMs with your own environment. For an example, take a look at [a sample environment](FIXME)

## Usage

To use this bosh release, first upload it to your bosh:

```
bosh target BOSH_HOST
git clone https://github.com/cloudfoundry-community/root-env-boshrelease.git
cd root-env-boshrelease
bosh upload release releases/root-env-1.yml

Or, alterntively, grab the latest final release from S3:

https://root-env-boshrelease.s3.amazonaws.com/boshrelease-root-env-1.tgz
```

For [bosh-lite](https://github.com/cloudfoundry/bosh-lite), you can quickly create a deployment manifest & deploy a cluster:

```
templates/make_manifest warden
bosh -n deploy
```

However that isn't terribly useful, and I don't recommend spinning up VMs using the root-env release's jobs for anything other
than testing your environment. The real power comes in when you apply the root-env job templates to VMs from other BOSH releases.

As an example, let's add it to the job spec for [cf-release](https://github.com/cloudfoundry/cf-release)'s  ```nats_z1``` job:

```
diff --git a/manifests/cf-manifest.yml b/manifests/cf-manifest.yml
index 0574b0a..9543ccb 100644
--- a/manifests/cf-manifest.yml
+++ b/manifests/cf-manifest.yml
@@ -131,30 +131,34 @@ jobs:
 - instances: 1
   name: nats_z1
   networks:
   - name: cf1
     static_ips:
     - 10.244.0.6
   properties:
     metron_agent:
       zone: z1
     networks:
       apps: cf1
+    root_env:
+      env_tarball: https://github.com/my-org/env-repo/archive/1.0.tar.gz
   resource_pool: medium_z1
   templates:
   - name: nats
     release: cf
   - name: nats_stream_forwarder
     release: cf
   - name: metron_agent
     release: cf
+  - name: root-env
+    release: root-env
   update: {}
```

**NOTE**

```root-env-boshrelease``` expects you to provide it with a URL for the source tarball of the root environment you want installed.
For this, I recommend github releases of an environment repo.

## make-manifest, etc

If you really want to deploy a single vm running the root-env job, feel free. ```templates/make_manifest <aws|warden>``` should get you started
