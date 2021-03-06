BOSH release for consul
=======================

Use this BOSH release to either:

-	deploy a cluster of consul servers; OR
-	upgrade an existing BOSH deployment to advertise or discover services

The [redis-boshrelease](https://github.com/cloudfoundry-community/redis-boshrelease) is an example BOSH release that can use this consul release to advertise itself to other consul consumers.

Installation
------------

This information was changed because of the switch to a new consul version.
Right now the consul tarball is downloaded via the update script.

```
<<<<<<< HEAD
bosh upload release https://consul-boshrelease.s3.amazonaws.com/boshrelease-consul-7.tgz
```

To deploy it you will need the source repository that contains templates:

```
git clone https://github.com/cloudfoundry-community/consul-boshrelease.git
cd consul-boshrelease
git checkout v8
```

=======
$> git clone https://github.com/cloudfoundry-community/consul-boshrelease.git
$> cd consul-boshrelease
$> ./update
$> bosh create release --force
$> bosh upload release
```
>>>>>>> anynines/master
Usage
-----

### First-time cluster deployment

For [bosh-lite](https://github.com/cloudfoundry/bosh-lite), you can quickly create a deployment manifest & deploy a 3-node cluster:

```
templates/make_manifest warden
bosh -n deploy
```

View the Consul UI on http://10.244.4.2:8500/ui

For AWS EC2, create a 3-node cluster:

```
templates/make_manifest aws-ec2
bosh -n deploy
```

### Upgrading cluster

The first time deployment will boot all VMs simultaneously. Afterwards we want to ensure at least one of the VMs is running at all times. So we need to change the manifest.

Running the `make_manifest` command again for an existing deployment will change the manifest.

```
templates/make_manifest warden
bosh -n deploy
```

Will include this change:

```
Jobs
consul_z1
  update
    ± max_in_flight:
      - 50
      + 1
```

Now onwards, each consul server node will be updated only when the other nodes are not being updated.

### Override security groups

For AWS & Openstack, the default deployment assumes there is a `default` security group. If you wish to use a different security group(s) then you can pass in additional configuration when running `make_manifest` above.

Create a file `my-networking.yml`:

```yaml
---
networks:
  - name: consul1
    type: dynamic
    cloud_properties:
      security_groups:
        - consul
```

Where `- consul` means you wish to use an existing security group called `consul`.

You now suffix this file path to the `make_manifest` command:

```
templates/make_manifest openstack-nova my-networking.yml
bosh -n deploy
```

Development
-----------

Requires Ruby 1.9+ for consul-ui's sass buildtool.

```
bundle install
git submodule update --init
bosh create release --force
```
