# Sia S3 Integration: rclone

## What is rclone?

Rclone is a command-line program to manage files on cloud storage. Rclone is very feature-rich and integrates with dozens of cloud storage providers, including any S3-compatible object stores like Sia `renterd`.

> Users call rclone _"The Swiss army knife of cloud storage"_, and _"Technology indistinguishable from magic"_.

Rclone [mounts](https://rclone.org/commands/rclone\_mount/) any local, cloud, or virtual filesystem as a disk on Windows, macOS, Linux, and FreeBSD and serves these over [SFTP](https://rclone.org/commands/rclone\_serve\_sftp/), [HTTP](https://rclone.org/commands/rclone\_serve\_http/), [WebDAV](https://rclone.org/commands/rclone\_serve\_webdav/), [FTP](https://rclone.org/commands/rclone\_serve\_ftp/), and [DLNA](https://rclone.org/commands/rclone\_serve\_dlna/). The mount lets us interact with our Sia `renterd` storage as a regular filesystem. We can mount `renterd` storage to a server’s filesystem or even a local laptop’s filesystem.

## Step 1: Enable `renterd` S3 API

Start by setting up `renterd` on your system. Follow the setup guide for your operating system.

{% hint style="info" %}
This guide requires a working install of `renterd`. If you have not already installed `renterd`, you will need to do so before continuing.

#### [📖 `renterd` Installation Guide](https://docs.sia.tech/v/current/renting/setting-up-renterd)
{% endhint %}

Before setting up `rclone` we will need to enable `renterd`'s S3 interface if you have not done so already. This is done by editing your `renterd.yml` file. If you are unsure where your `renterd.yml` is installed, please refer to the correct [installation guide](https://docs.sia.tech/v/current/renting/setting-up-renterd) for your system.

Once you have located your `renterd.yml` file, open it with your preferred text editor, and add the following:

```yaml
s3:
  enabled: true
  address: localhost:9985
  keypairsV4:
    your_renterd_access_key: your_renterd_secret_key
```

Make sure to replace `your_renterd_access_key` and `your_renterd_secret_key` with unique passphrases. They can be anything you want but must be at least 16 characters in length, and your secret key should always be kept private. Keep these values on hand. You will need them for Step 3.

{% hint style="warning" %}
If you are running `renterd` in a docker container, you will need to override the address via docker: `command: -s3.address:9985`
{% endhint %}

## Step 2: Install rclone

{% tabs %}
{% tab title="Windows" %}
1. Press `windows key + R` to open the run dialog. Type in `powershell` and press `OK` to open a Terminal.

<div data-full-width="false">

<figure><img src=".gitbook/assets/rclone-new-config-win-01 (10).png" alt="" width="375"><figcaption></figcaption></figure>

</div>

2. Once the Terminal loads, run the following command to install `rclone`.

```powershell
winget install Rclone.Rclone
```

<figure><img src=".gitbook/assets/rclone-new-config-win-02 (10).png" alt=""><figcaption></figcaption></figure>
{% endtab %}

{% tab title="Mac OS" %}
1.  Press `CMD + Space` to open Spotlight search and open a`terminal`.

    <figure><img src=".gitbook/assets/rclone-new-config-macos-01.png" alt="" width="563"><figcaption></figcaption></figure>
2. Once the Terminal loads, run the following command to install `rclone`.

{% code fullWidth="false" %}
```shell-session
sudo -v ; curl https://rclone.org/install.sh | sudo bash
```
{% endcode %}

<figure><img src=".gitbook/assets/rclone-new-config-macos-02.png" alt=""><figcaption></figcaption></figure>
{% endtab %}

{% tab title="Linux" %}
1. Press `Ctrl + Alt + T` to open a new `Terminal` window and run the following command to install `rclone`.

<pre class="language-console"><code class="lang-console"><strong>sudo apt install rclone
</strong></code></pre>

{% hint style="info" %}
If you are unable to open a `Terminal` using the above method, try one of the other methods [listed here](https://www.geeksforgeeks.org/how-to-open-terminal-in-linux/).
{% endhint %}

<figure><img src=".gitbook/assets/rclone-new-config-linux-01.png" alt=""><figcaption></figcaption></figure>
{% endtab %}
{% endtabs %}

## Step 3: Configuring rclone

Now that we have `rclone` installed, we can use the interactive configuration wizard to set up a new remote. To do so, run the following command from the Terminal.

```shell-session
rclone config
```

{% hint style="info" %}
If the command does not work, you may need to restart your terminal first. To do so close the current `Terminal` and use the same method you used abov e in Step 2 to open a new one.
{% endhint %}

When the configuration wizard loads, enter `n` to create a new remote.

```shell-session
n) New remote
s) Set configuration password
q) Quit config
n/s/q> n
```

Next, name your remote. You can name it anything you want, but for this guide, we will be naming it `renterd`.

```shell-session
Enter name for new remote.
name> renterd
```

You will now be given a list of `Storage` options. Type in `s3` and press `Enter`.

```shell-session
Option Storage.
Type of storage to configure.
Choose a number from below, or type in your own value.
...
Storage> s3
```

Next, you will be asked to select a `Provider`. Type in `other` and press `Enter`

```shell-session
Option provider.
Choose your S3 provider.
Choose a number from below, or type in your own value.
...
provider> other
```

You will now be asked to select how you would like to supply your S3 credentials. Since we are supplying a `access_key_id` and `secret_access_key` we will not be using environment variables. Type in `false` or simply press `Enter` to use the default.

```
Option env_auth.
Get AWS credentials from runtime (environment variables or EC2/ECS meta data if no env vars).
Only applies if access_key_id and secret_access_key is blank.
Choose a number from below, or type in your own boolean value (true or false).
Press Enter for the default (false).
...
env_auth> false
```

For the next two questions, you will be prompted for your `access_key_id` and `secret_access_key` that you set in Step 1.

```
Option access_key_id.
AWS Access Key ID.
Leave blank for anonymous access or runtime credentials.
Enter a value. Press Enter to leave empty.
access_key_id> your_renterd_access_key
```

```
Option secret_access_key.
AWS Secret Access Key (password).
Leave blank for anonymous access or runtime credentials.
Enter a value. Press Enter to leave empty.
secret_access_key> your_renterd_secret_key
```

When prompted for your `Region` press `Enter` to leave it blank.

```
Option region.
Region to connect to.
Leave blank if you are using an S3 clone and you don't have a region.
Choose a number from below, or type in your own value.
Press Enter to leave empty.
...
region>
```

You will now be asked to enter an `Endpoint`. This should be the same as the `address` parameter we configured in our `renterd.yml` for Step 1.

```
Option endpoint.
Endpoint for S3 API.
Required when using an S3 clone.
Enter a value. Press Enter to leave empty.
endpoint> localhost:9985
```

When asked for a `Location constraint` press `Enter` to leave it empty.

```
Option location_constraint.
Location constraint - must be set to match the Region.
Leave blank if not sure. Used when creating buckets only.
Enter a value. Press Enter to leave empty.
location_constraint>
```

Next, you will be asked to select an `ACL` from the list provided. Type in `private` and press `Enter`.

```
Option acl.
Canned ACL used when creating buckets and storing or copying objects.
This ACL is used for creating objects and if bucket_acl isn't set, for creating buckets too.
For more info visit https://docs.aws.amazon.com/AmazonS3/latest/dev/acl-overview.html#canned-acl
Note that this ACL is applied when server-side copying objects as S3
doesn't copy the ACL from the source but rather writes a fresh one.
If the acl is an empty string then no X-Amz-Acl: header is added and
the default (private) will be used.
Choose a number from below, or type in your own value.
Press Enter to leave empty.
...
acl> private
```

When asked if you would like to edit the advanced config, type in `n` and press `Enter`.

```
Edit advanced config?
y) Yes
n) No (default)
y/n> n
```

You will now be given a summary of your new remote. If they are correct you can type in `y` and press `Enter` to save your remote.

```
Configuration complete.
Options:
- type: s3
- provider: other
- access_key_id: your_renterd_access_key
- secret_access_key: your_renterd_secret_key
- endpoint: localhost:9985
- acl: private
Keep this "renterd" remote?
y) Yes this is OK (default)
e) Edit this remote
d) Delete this remote
y/e/d> y
```

You have now successfully created a remote for `renterd`. You can now type in `q` and press `Enter` to quit the configuration wizard.

## Step 4: Mount the filesystem

{% hint style="warning" %}
This tutorial is focused on Linux. For Windows and MacOS, please visit the official documentation: [https://rclone.org/commands/rclone\_mount/](https://rclone.org/commands/rclone\_mount/)
{% endhint %}

Now that `renterd` is running and `rclone` is configured with `renterd` as a remote, we can mount the `renterd` storage to the filesystem.

{% tabs %}
{% tab title="Windows" %}
To run rclone mount on Windows, you will first need to download and install [WinFsp](https://winfsp.dev/rel/).

Once you have installed `WinFsp`, you can then mount your `renterd` remote and assign it the drive letter `X:` using the following command.

```
rclone mount renterd:/default X: --vfs-cache-mode full
```

{% hint style="info" %}
If you would like to run `rclone` as a background service so you can use the `--daemon` flag.
{% endhint %}

If you have configured everything properly, you should see a confirmation that `rclone` has successfully started.

<figure><img src=".gitbook/assets/rclone-setup-win-mount-01.png" alt=""><figcaption><p>Mounting the <code>renterd</code> remote with <code>rclone</code>.</p></figcaption></figure>

To confirm you have mounted your Sia storage correctly, you should see a new `X:` drive on your filesystem.

<figure><img src=".gitbook/assets/rclone-setup-win-mount-02.png" alt=""><figcaption><p>Sia storage mounted as <code>X:</code> drive.</p></figcaption></figure>

You can now access your files on Sia directly from your File Explorer.

<figure><img src=".gitbook/assets/rclone-setup-win-mount-03.png" alt=""><figcaption><p>View your files directly from File Explorer.</p></figcaption></figure>

<figure><img src=".gitbook/assets/rclone-setup-win-mount-05 (1).png" alt=""><figcaption><p>Your file as seen in the <code>renterd</code> Web UI.</p></figcaption></figure>
{% endtab %}

{% tab title="Mac OS" %}
Start by creating an empty directory on your filesystem that we will use as the mount point. Once the mount is set up, this is where all the files will appear.

```shell-session
mkdir ~/root/files
```

Next, create the mount. Adding the `--daemon` flag runs the mount in the background.

```shell-session
rclone mount renterd:/default /mnt/files \
  --allow-other \
  --vfs-cache-mode full \
  --daemon
```

To later unmount the remote:

```shell-session
fusermount -u /mnt/files
```

You should now be able to navigate your files as a filesystem. Files copied into the directory tree will be stored on Sia.

```shell-session
$ ls /mnt/files/
Screenshot.png
```
{% endtab %}

{% tab title="Linux" %}
Start by creating an empty directory on your filesystem that we will use as the mount point. Once the mount is set up, this is where all the files will appear.

```shell-session
mkdir /mnt/renterd_files
```

Next, create the mount. Adding the `--daemon` flag runs the mount in the background.

```shell-session
rclone mount renterd:/default /mnt/renterd_files \
  --allow-other \
  --vfs-cache-mode full \
  --daemon
```

To later unmount the remote:

```shell-session
fusermount -u /mnt/renterd_files
```

You should now be able to navigate your files as a filesystem. Files copied into the directory tree will be stored on Sia.

```shell-session
$ ls /mnt/renterd_files
Screenshot.png
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
For more details and system-specific instructions, visit the official rclone mount documentation: [https://rclone.org/commands/rclone\_mount](https://rclone.org/commands/rclone\_mount/)
{% endhint %}

## All done.

We used `renterd` and rclone to mount our Sia storage as a filesystem. This is a great way to use Sia for your files, especially for use cases such as large video or media libraries that take up many terabytes of space but you would still like available for streaming at any moment. If this guide has been engaging, check out our website to read more about Sia and `renterd`, and join our Discord, where the team and community can answer your questions!
