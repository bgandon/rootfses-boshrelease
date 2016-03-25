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

3. Staying in the `packages/cflinuxfs2`, delete any previous version of the
   downloaded `cflinuxfs2-1.*.0.tar.gz` archive.

        rm -r cflinuxfs2/

4. Staying in the `packages/cflinuxfs2`, run the `prepare` script.

        ./prepare

5. Add the blob to your currently-being-built release, using the `cflinuxfs2/`
   path in the blobstore so that it matches the specified path in `spec`,
   which will be updated in the next step.

        cd ../..
        bosh add blob packages/cflinuxfs2/cflinuxfs2/cflinuxfs2-1.48.0.tar.gz cflinuxfs2

6. Edit the `spec` for the `cflinuxfs2` package to properly reference the blob
   you just added.

        files:
          - cflinuxfs2/cflinuxfs2-1.48.0.tar.gz

7. Edit the `packaging` script in `packages/cflinuxfs2` and again change the
   version line as in the `prepare` script.

        version="1.48.0"

8. Commit all your changes to git, tag them `v1.48.0`, and push all that
   (including the tag) to a repo of your choice (public or private, but your
   BOSH Director need to be able to access it if you specified the `rootfses`
   release as a git repo in your deployment)

        git commit -am "Added blob for cflinuxfs2 version 1.48.0 + Updated blob version in package + Updated scripts"
        git push


9. Upload your blobs to the blobstore. Here it just copies the new blob to the
   local blobstore in `./local_blobstore`.

        bosh upload blobs

10. At the root of the release directory, create a dev release to test the
    result, using the same version as in `prepare` and `packaging` above.

        cd ../..
        bosh create release --with-tarball --name rootfses --version 1.48.0

11. Upload your dev created release to your BOSH director.

        bosh upload release dev_releases/rootfses/rootfses-1.48.0.tgz

12. Update your diego deployment to use the new `roofs` version you just
    uploaded.

        releases:
          - name: rootfses
            version: 1.48.0

13. Test deployment with `bosh deploy`. If it fails, find the problem and when
    you know what to fix, reset your release with `bosh reset release`, make
    your fixes. Then rollback your deployment manifest, deploy it back with
    `bosh deploy` and when the dev release is no more used by any deployment
    (which you can check with `bosh releases`) then delete it with
    `bosh -n delete release rootfses 1.48.0` and go back to step #10.

14. Create a final release out of your dev release.

        bosh -n create release --final --name rootfses --version 1.48.0

15. Commit your final release to git and tag it properly.

        git add .
        git commit -m "Created final 'rootfses' release version 1.48.0"
        git tag v1.48.0
        git push
        git push --tags

Have more fun!
