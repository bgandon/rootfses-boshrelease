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

4. Check out the latest version of this release:

        ls releases/rootfses/rootfses-*.yml

5. Edit `config/final.yml` and update the `blobstore_path` so that it matches
   a local directory for your blobs, like the `./local_blobstore` I use myself.

6. If the latest version is `1.43.0`, create the release tarball and upload it
   to the BOSH director like this:

        bosh create release --final --name rootfses --version 1.43.0
        bosh upload release

7. Deploy with `bosh deploy`.

Have fun!


How to create a new version of this release?
--------------------------------------------

If a [new version of cflinuxfs2](https://github.com/cloudfoundry/stacks/releases)
is released, let's say `1.48.0`, then

1. Edit the `packages/cflinuxfs2/prepare` script in your favorite editor.
   Let's say Emacs because Emacs rocks.

        cd packages/cflinuxfs2
        emacs prepare

2. Update the version line like this.

        version="1.48.0"

3. Staying in the `packages/cflinuxfs2`, run the `prepare` script.

        ./prepare

4. Edit the `packaging` script in `packages/cflinuxfs2` and again change the
   version line as in the `prepare` script.

        version="1.48.0"

5. Commit your changes to git.

6. At the root of the release directory, create the final release using the
   same version as in `prepare` and `packaging` above.

        cd ../..
        bosh create release --final --name rootfses --version 1.48.0

7. Staying in this directory, upload your newly created release to your BOSH
   director.

        bosh upload release

8. Deploy with `bosh deploy`.

Have more fun!

