# Notes for distributors

## Importing images

Images of Container Linux alpha releases are hosted at `https://alpha.release.core-os.net/amd64-usr/`. There are directories for releases by version as well as `current` with a copy of the latest version. Similarly, beta releases can be found at `https://beta.release.core-os.net/amd64-usr/`.

If you are importing images for use inside of your environment it is recommended that you import from the `current` directory. For example to grab the alpha OpenStack version of Container Linux you can import `https://alpha.release.core-os.net/amd64-usr/current/coreos_production_openstack_image.img.bz2`. There is a `version.txt` file in this directory which you can use to label the image with a version number.

It is recommended that you also verify files using the [CoreOS Image Signing Key][signing-key]. The GPG signature for each image is a detached `.sig` file that must be passed to `gpg --verify`. For example:

```sh
wget https://alpha.release.core-os.net/amd64-usr/current/coreos_production_openstack_image.img.bz2
wget https://alpha.release.core-os.net/amd64-usr/current/coreos_production_openstack_image.img.bz2.sig
gpg --verify coreos_production_openstack_image.img.bz2.sig
```

[signing-key]: https://coreos.com/security/image-signing-key

## Image customization

Customizing a Container Linux image for a specific operating environment is easily done through [cloud-config](https://github.com/coreos/coreos-cloudinit/blob/master/Documentation/cloud-config.md/), a YAML-based configuration standard that is widely supported. As a provider, you must ensure that your platform makes this data available to Container Linux, where it will be parsed during the boot process.

Use cloud-config to handle platform specific configuration such as custom networking, running an agent on the machine or injecting files onto disk. Container Linux will automatically parse and execute `/usr/share/oem/cloud-config.yml` if it exists. Your cloud-config should create additional units that process user-provided metadata, as described below.

## Handling end-user cloud-config files

End-users should be able to provide a cloud-config file to your platform while specifying their VM's parameters. This file should be made available to Container Linux at a known network address, injected directly onto disk or contained within a [config-drive][config-drive-docs]. Below are a few examples of how this process works on a few different providers.

[config-drive-docs]: http://docs.openstack.org/user-guide/cli_config_drive.html

### Amazon EC2 example

Container Linux machines running on Amazon EC2 utilize a two-step cloud-config process. First, a cloud-config file baked into the image runs systemd units that execute scripts to fetch the user-provided SSH key and fetch the [user-provided cloud-config][amazon-cloud-config] from the instance [user-data service][amazon-user-data-doc] on Amazon's internal network. Afterwards, the user-provided cloud-config, specified from either the web console or API, is parsed.

You can find the [code for this process on GitHub][amazon-github]. End-user instructions for this process can be found on our [Amazon EC2 docs][amazon-cloud-config].

[amazon-github]: https://github.com/coreos/coreos-overlay/tree/master/coreos-base/oem-ec2-compat
[amazon-user-data-doc]: http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AESDG-chapter-instancedata.html#instancedata-user-data-retrieval
[amazon-cloud-config]: booting-on-ec2.md#cloud-config

### Rackspace Cloud example

Rackspace passes configuration data to a VM by mounting [config-drive][config-drive-docs], a special configuration drive containing machine-specific data, to the machine. Like, Amazon EC2, Container Linux images for Rackspace contain a cloud-config file baked into the image that runs units to read from the config-drive. If a user-provided cloud-config file is found, it is parsed.

You can find the [code for this process on GitHub][rackspace-github]. End-user instructions for this process can be found on our [Rackspace docs][rackspace-cloud-config].

[rackspace-github]: https://github.com/coreos/coreos-overlay/tree/master/coreos-base/oem-rackspace
[rackspace-cloud-config]: booting-on-rackspace.md#cloud-config
