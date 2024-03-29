## Important disclaimer for testers
We appreciate your time testing Bunkr. Please note that during this testing phase, we ask you not to use Bunkr for any production scenarios or to store any information that may cause you problems if lost or exposed. You should assume that as part of our testing, we may delete all of the information that you keep inside.

Bunkr's beta is intended to gather feedback on the usage and possibilities of an application for the custody of a vast range of secrets. We know it is still rough around the edges but we would like to get to know what you think of our big picture.

For any questions, ideas or problems, please use GitHub to open an Issue within this repository.

We have been working really hard for a long time to bring Bunkr to you and we are very excited that this is happening. 

Thanks a lot for your support!

### What is the connection between Blink and Bunkr?
Bunkr and Blink are separated companies and teams. The original idea for Bunkr actually preceded Blink, and it was about handling secrets within development teams. Although there are now some solutions, we think they are all complex to use or not adecquate. We see Bunkr as not just a solution to this original problem, but going beyond in a world of other possibilities too.

# What is Bunkr

Bunkr is a command line interface and accompanying mobile application for secure storage and dynamic access management of sensitive or private data. Bunkr users store their private data across a number of remote machines using state of the art cryptographic techniques that make sure the data is maximally resistant to either theft or loss. Data stored in Bunkr is accessible only to the exact devices specified by the user(s), and permissions can be granted and revoked dynamically. Even the remote machines that store and serve data to Bunkr users have no way of knowing the underlying data they store. Files, Passwords, API Keys, Cryptographic Keys and more can all be handled with previously unimaginable security and convenience thanks to Bunkr.

## Bunkr App (mobile)

The current mobile app has been stripped down to showcase the functionality of signing with an SSH key from Blink Shell. For extended functionality, we recommend to use the CLI for Bunkr. 

Please make sure both Blink Shell and Bunkr are installed on the same device. Blink requires a new TestFlight version which includes the `bunkr` command. Bunkr requires to sign in to verify the identity of the Device.

### Creating and using a Bunkr key within Blink

To create a Bunkr SSH key, use the `bunkr keygen <name>` command within Blink. This will give instructions to Bunkr to create a secret shared ECDSA-256 key that can only currently be accessed by the client it was created.

To use this key for login, first copy the Public Key to `.ssh/authorized_keys` on the host you would like to access. You can find the Public Key on Blink within config > Keys > <name>. *Please Note*, do not try to copy the private key as it is currently broken.

Subsequently, run `ssh-agent` in Blink. After that you should be good to connect to the server by running `ssh user@host` or `mosh user@host`. You should see Blink forwarding the signature request to the Bunkr app to request your permission for signing, and upon success, you will be logged in.

The magic within Bunkr makes it so that the Bunkr app also didn't perform the signature. Your signature was completed by a coalition of machines, each only holding a share of your private key, *and without reconstructing the original key on any device or on any computer process*.

### Sharing a Private Key with other devices

Bunkr makes it possible to use a single Private Key *from* multiple devices in a secure way. As the Private Key is never recomposed during signature, you can only grant signature capabilities to other devices and still be in control of the key. This greatly simplifies Key Management by displacing it from the servers into Bunkr. If you lose one of your devices, just revoke the Signature capability of that device in your Bunkr instead of changing configuration in all your servers and services.

To share a key with other devices, first you need to [obtain the Identity of those devices](#send-device). Then within Bunkr app Open the Key by tapping on it. Within the Capabilities section you should see a single capability for the current device. Tap on /Add/ to grant a capability to the other device. A second Capability should now show up within the Secret screen, and you have to manually send it to the new device. To do so, tap on the new Capability and the iOS Sharing Sheet will pop out. *Please Note* that capabilities are attached to the Identity of a device, so they are safe to share over messages, email, etc...

To receive the capability on the other device:
* On iOS, it will happen automatically by just opening the link. 
* Within the CLI, please check out [receive-capability](#receive-capability).

You can see how easy it is to control access by just tapping on the Grant/Revoke button.

*Please Note* that at the moment, unless a secret is shared by multiple devices, losing your device will effectively lose your secrets. This will be solved in the future.

The rest of this document will focus only on the Bunkr CLI.

## Bunkr CLI

### Installation and Setup

To install the Bunkr CLI you first must download the beta version as a binary file [here](https://github.com/off-the-grid-inc/bunkr-cli/releases).

We recommend to mv the binary to /usr/local/bin or corresponding folder. After installation, execute the command `$ bunkr -I` to run an interactive session.

If you see a `bunkr >>` prompt you are successfully running the Bunkr CLI in interactive mode. The rest of this document will deal with the commands that can be executed from this interactive session. Executing commands straight from the command line in one shot, and running a demonized version of Bunkr in the background will be dealt with only at the end of this document.

### Registering a Device

There is a one time sign in process to authorize your device to use Bunkr. Before any other commands can be executed the CLI will prompt the user to complete this process:

1. Request to register the device. You’ll need to provide a valid email address and a name for your device: `>>> signin <email> <device name>;`
2. A verification code will be sent to your email. Go to your email and copy the code.
3. Finish registering the device with: `confirm-signin <email> <verification code>;`

## Creating and managing Secrets

### new-text-secret

To store a basic text secret, like a password, execute:
 
`>>> new-text-secret <secret name> <text>;`

The secret name is just a reference name for your stored secret data. The text can be any set of utf-8 symbols. If you want to include spaces in the command arguments then frame them with quotes. For example:

`>>> new-text-secret "API TOKEN" "zxcvasdfqwer!@#$1234";`

### new-ssh-key 

To store an ssh key (type ecdsa, nist-p256 curve), execute:

`>>> new-ssh-key <secret name>;`

Upon success the ssh key stored in murmur will automatically be added to the Bunkr SSH Agent keyring, and the associated public key will be output in the terminal. You can copy and save the public key to a file or add it to the authorized_keys of desired servers.

If the device has ssh configured to forward authentication to the Bunkr Agent, We can sign the authentication request with Bunkr, which means we can get the signature and authenticate our device with the desired remote machine, *without ever exposing the private key on our machine or on any computer process whatsoever*. This provides an unmatched level of security for managing access to remote servers.

### new-secret-from-file

To store a local file directly into Bunkr (the size limit is currently 1MB), execute:

`>>> new-secret-from-file <secret name> <path to file>;`

The raw binary data of the file will now be stored in Bunkr.

### import-ssh-key

If one already has a PEM encoded ecdsa-p256 private key file and wants to store that file in the Bunkr SSH Agent keyring, execute:

`>>> import-ssh-key <secret name> <path to private key file>;`

### access

The access command recomposes the data securely stored in Bunkr on your local machine. The access command will only succeed if the device requesting access actually has permission to do so. To access, execute:

`>>> access <secret name> (optional: <mode>);`

The optional mode argument specifies how to output the retrieved data. There are three options:

- `b64` which will return the data as a base64 encoded string
- `text` which assumes the data to be text and encodes it as a utf-8 string.
- `file <filepath>` which will write the data as a file (and saved to the path specified).

### create

The create command is a low level command which should only be used by experienced users. It creates a new namespace in Bunkr without writing any data to Bunkr yet. Execute:

`>>> create <secret name> <secret type>;`

The possible secret types are: 
- GENERIC-GF256 (used for text)
- ECDSA-P256 (used for ssh keys)
- ECDSA-SECP256k1 (used for bitcoin keys)

### write

The write command is used to write (or overwrite) data to a Bunkr Secret. Execute:

`>>> write <secret name> <mode> <data>;`

The possible mode and data combinations are:
- `b64 <base 64 encoded data>`
- `text <utf-8 string>`

As an example we can use the create and write commands to store a Bitcoin Private Key in Bunkr:

```
>>> create myBitcoinKey ECDSA-SECP256k1;
>>> write myBitcoinKey b64 3ABFM/7485746283498BFAC/2==;
```

### delete

To delete a Bunkr Secret execute:

`>>> delete <secret name>;`

This command completely removes a secret from Bunkr. This includes all information about permissions, and any secret data. This operation is only permitted for an Admin on the secret

## Sharing secrets between devices

### send-device

To send your device information to other users, execute:
`>>> send-device (optional: <device name>);`

This will output a link which you can send by any means (sms, email etc.) to another desired Bunkr user. The optional argument is to give your device a custom name. For example:

`>>> send-device “John’s Work Computer”;`

### receive-device

To store information about a third party device after receiving a device link simply execute:

`>>> receive-device < url > (optional: <custom device name>);`

This will store the device information locally, under the provided device name (the optional parameter). If no name is provided it will default to the name the user gave their device on sending it.

### grant

When creating a new Bunker secret the device used, and that device alone, has full permissions on the secret. It can access the data, overwrite it, and change or extend the permissions arbitrarily. To grant permission for any other device on a secret you’ll first have to know other devices (see: receive-device). Then you can execute:

`>>> grant <device name> <secret name> (optional: <admin flag>);`

Here you are granting permission to the specified device on the specified secret. The optional admin flag (just type `admin`) specifies whether the permission is admin level or not. Admin level permissions means that this new device will also be able to alter permissions and overwrite the secret. Without the admin flag the permission is access-only (in the case of text secrets) or to signature-only (in the case of ssh keys). Here is an example of granting admin capabilities:

`>>> grant “my iPhone” myBitcoinKey admin;`

Now the device “my iPhone” would be able to query, overwrite, and add permissions to the secret myBitcoinKey. If you are not an admin on a secret then a grant operation will fail (you are not authorized to change the permissions, so the operation will not be executed)

### revoke

If a device has been compromised, no longer exists, or you simply don’t want it to have capabilities on a secret anymore you can revoke their access. Execute:

`>>> revoke <device name> <secret name>;`

This completely removes that devices permissions on that resource. If you are not an admin on a secret then the revoke operation will fail. WARNING: Do not revoke yourself from a secret if you are the only admin. This could potentially lock you out of your own secret completely and make the data irretrievable.

### send-capability

Once you have granted a device some level of permission on a secret (known as a capability), the device cannot use the capability until they know about it. 

`>>> send-capability <device name> <secret name>;`

Provided the device actually has been granted some abilities on the secret, this command will output a short link which can be sent over any ordinary channel (sms, email etc.) to the desired party. When the party receives the link they can import their capability information and start using the secret.

### receive-capability

If a third party device has given you capability on one of their secrets you will need to import that capability. Execute:

`>>> receive-capability < url > (optional: <custom secret name>);`

## Bunkr CLI config

### One Shot commands

All the commands described above can be executed directly from the terminal (not in interactive mode) by simply prepending `./bunkr` to every command. For example:

`$ ./bunkr new-text-secret mySecret /"this is my secret/";`

notice that in some shells you may need escape characters for the quotations to work.

## Demo Use Cases for Bunkr CLI

### Quick Demo of Bunkr SSH Agent

In another terminal run:
```
$ cat <path to ssh public key> >> ~/.ssh/authorized_keys
$ export SSH_AUTH_SOCK=/tmp/agent.sock
$ ssh localhost
```

It worked if you don't get asked for a password and the bunkr cli terminal has a log that says `signing with murmur...` (if you don't get asked for a password but you don't read that log then you signed with another key you already have configured in `~/.ssh`)

### Use Bunkr SSH Agent for Github Pushes

1. Create a new Bunkr SSH Key for your desktop.
2. Take the contents of public key and on github make a new SSH key [here](https://www.github.com/settings/keys)
3. Point your remote to `git@github.com:<name-of-acct>/<name-of-repo>`
4. To push or pull to that repository you will now sign with bunkr. Will only succeed if you are running the bunkr agent (you can stop and restart without issues, to restart just do `./bunkr-cli -I`) AND you have the right env var for the socket (`export SSH_AUTH_SOCK=/tmp/agent.sock`)

