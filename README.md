Rootfses BOSH Release
=====================

This [BOSH release](http://bosh.io/docs/release.html) helps you in deploying
new versions of root filesystems for [Cloud Foundry](https://github.com/cloudfoundry/cf-release),
and especially the one and only
[cflinuxfs2](https://github.com/cloudfoundry/stacks) (misnamed “stacks” in the
Cloud Foundry parlance).

In a Cloud Foundry deployment, this “cflinuxfs2” root filesystem is the basis
for all containers where applications are deployed using
[buildpacks](http://docs.gstack.io/app-and-context/#a-buildpack).

So you need to update it.

And currently, there is one new version available every 3 days. Yes, *two* new
version per week.

And the best part is: there is no easy way to update “cflinuxfs2”, except this
very BOSH relsease. That's why you definitely need it.


Prerequisites
-------------

Only [Diego](https://github.com/cloudfoundry-incubator/diego-release) is
supported. Diego rocks anyway. You shall upgrade to Diego.


Usage
-----

Here are the basic instruction

1. Patch Diego deployment manifests with the
   [deployment-samples/diego-manifests.yml.patch](./deployment-samples/diego-manifests.yml.patch)

2. Add the [deployment-samples/property-overrides.yml](./deployment-samples/property-overrides.yml)
   to the Diego deployment

3. If needed, customize the [deployment-samples/rootfses-properties.yml](./deployment-samples/rootfses-properties.yml)
   and add them to your Diego deployment.

4. Create the release tarball and upload it to the director:

        bosh create release --final --name rootfses --version 1.43.0
        bosh upload release

5. Deploy with `bosh deploy`.

Have fun!