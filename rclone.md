# rclone

## Sia S3 Integration: rclone

### What is rclone?

Rclone is a command-line program to manage files on cloud storage. Rclone is very feature-rich and integrates with dozens of cloud storage providers, including any S3-compatible object stores like Sia `renterd`.

> Users call rclone _"The Swiss army knife of cloud storage"_, and _"Technology indistinguishable from magic"_.

Rclone [mounts](https://rclone.org/commands/rclone\_mount/) any local, cloud, or virtual filesystem as a disk on Windows, macOS, Linux, and FreeBSD and serves these over [SFTP](https://rclone.org/commands/rclone\_serve\_sftp/), [HTTP](https://rclone.org/commands/rclone\_serve\_http/), [WebDAV](https://rclone.org/commands/rclone\_serve\_webdav/), [FTP](https://rclone.org/commands/rclone\_serve\_ftp/), and [DLNA](https://rclone.org/commands/rclone\_serve\_dlna/). The mount lets us interact with our Sia `renterd` storage as a regular filesystem. We can mount `renterd` storage to a server’s filesystem or even a local laptop’s filesystem.

### Step 1: Enable `renterd` S3 API

Start by setting up `renterd` on your system. Follow the setup guide for your operating system.

{% hint style="info" %}
This guide requires a working install of `renterd`. If you have not already installed `renterd`, you will need to do so before continuing.

#### [📖 `renterd` Installation Guide](https://docs.sia.tech/v/current/renting/setting-up-renterd)
{% endhint %}

Before setting up `rclone` we will need to enable `renterd`'s S3 interface if you have not done so already. This is done by editing your `renterd.yml` file. If you are unsure where your `renterd.yml` is installed, please refer to the correct [installation guide](https://docs.sia.tech/v/current/renting/setting-up-renterd) for your system.

Once you have located your `renterd.yml` file, open it with your preferred text editor and add the following:

```yaml
s3:
  enabled: true
  address: localhost:9985
  keypairsV4:
    your_renterd_access_key: your_renterd_secret_key
```

Make sure to replace `your_renterd_access_key` and `your_renterd_secret_key` with unique pass phrases. They can be anything you want, but must be at least 16 characters in length, and your secret key should always be kept private. Keep these values on hand, you will need them for Step 3.

{% hint style="warning" %}
If you are running `renterd` in a docker container, you will need to override the address via docker: `command: -s3.address:9985`
{% endhint %}

### Step 2: Install rclone

[Download and install](https://rclone.org/downloads/) the correct version of `rclone` for your system.

{% hint style="info" %}
If your system is running Linux you can install `rclone` using the `apt` package manager with the following command:

```console
sudo apt install rclone
```
{% endhint %}

### Step 3: Configure rclone

Next, we must add our `renterd` node as remote storage to rclone. We will do this by running the `rclone` interactive configuration wizard from a terminal.

{% tabs %}
{% tab title="Windows" %}
{% hint style="info" %}
You will need to open a command prompt and `cd` to the location where you have `rclone.exe`. Alternatively you can add `rclone.exe` to your environment variables and run it from any location.
{% endhint %}

Press `windows key + R` to open the run dialog.

```console
rclone.exe config
```
{% endtab %}

{% tab title="MacOS/Linux" %}
```console
rclone config
```
{% endtab %}
{% endtabs %}

```toml
[renterd] # name of remote
type = s3
provider = Other
access_key_id = renterd-access-key # your access key
secret_access_key = renterd-secret-key # your secret key
endpoint = http://localhost:9985 # s3 address from your renterd config
acl = private
```

### Step 4: Mount the filesystem

{% hint style="warning" %}
This tutorial is focused on Linux. For Windows and MacOS, please visit the official documentation: [https://rclone.org/commands/rclone\_mount/](https://rclone.org/commands/rclone\_mount/)
{% endhint %}

Now that `renterd` is running and `rclone` is configured with `renterd` as a remote, we can mount the `renterd` storage to the filesystem.

Start by creating an empty directory on your filesystem that we will use as the mount point. Once the mount is set up, this is where all the files will appear.

```console
mkdir ~/root/files
```

Next, create the mount. Adding the `--daemon` flag runs the mount in the background.

```console
rclone mount renterd:/default /mnt/files \
  --allow-other \
  --vfs-cache-mode full \
  --daemon
```

To later unmount the remote:

```console
fusermount -u /mnt/files
```

For more details and system-specific instructions, visit the official rclone mount documentation: [https://rclone.org/commands/rclone\_mount/](https://rclone.org/commands/rclone\_mount/)

You should now be able to navigate your files as a filesystem. Files copied into the directory tree will be stored on Sia.

```console
$ ls /mnt/files/
Screenshot.png
```

## All done.

We used `renterd` and rclone to mount our Sia storage as a filesystem. This is a great way to use Sia for your files, especially for use cases such as large video or media libraries that take up many terabytes of space but you would still like available for streaming at any moment. If this guide has been engaging, check out our website to read more about Sia and `renterd`, and join our Discord, where the team and community can answer your questions!
