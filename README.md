# wpkg: the web's package manager

> **Note: this software does not yet exist. Everything below is for preliminary
> design purposes only**

`wpkg` is a package manager for the web. It makes it easy to install packages
from untrusted sources while still ensuring that those packages contain the
data their maintainer intended them to.

It does this through the use of public key cryptography and visible maintainer
user names. To explain, lets look at an example installation command:
```
wpkg install vitiral:23ad2323ef322343de32a \
    https://github.com/vitiral/artifact/blob/releases.json
```

Let's break this down:
- `wpkg install`: this is the installation command
- `vitiral:<public key>`: this is the maintainer "vitiral" followed
  by the their public key, copied from the package's installation
  instructions.
- `https://...releases.json`: this is the path to a releases file which
    simply contains a list of versions and the url each platform
    can be downloaded from.

wpkg will download the tarfile and unpack it. Its contents will look
like:
```
artifact-app-0.9.2-x86_64-unknown-linux-gnu/
  +- art
  +- package.json
```

`art` is the downloaded binary and `package.json` is a json file containing the
following information:
```
{
    "metadata": {
        "name": "artifact",
        "maintainer": "vitiral",
        "version": "1.2.5",
        "license": "LGPLv3",
        "issueUrl": "http://github.com/vitiral/artifact/issues",
        "documentationUrl": "http://github.com/vitiral/artifact",
        "metadataVersion": "1"
    },
    "binaries": [
        {
            "name": "art",
            "path": "./art",
            "signature": "<signature of vitiral+$(cat ./art) file>",
            "signatureType": "SHA256"
        }
    ]
}
```

> Note: this file will almost surely be extended in the future.
> In particular, documentation files (both man and web-based) as
> well as dependencies will eventually be supported.

wpkg will then join the username and binary data of `./art` and
verify it with the public key and signature.

It will also cache `vitiral:<public key>` and the url of the `releases.json`.
If the user wants to upgrade `artifact` they only need to run:

```
wpkg upgrade artifact
```

It will use the cached `releases.json` web location to get the latest release
and will download it and validate it using the cached public key. The user
can also specify a specific version to upgrade and downgrade to.

Binaries will be installed in `$WPKG_HOME/bin` and the installation script for
wpkg will add this to the `PATH` of `.bashrc`, etc files.

`$WPKG_HOME` will be located in `/.local/wpkg` by default.

# Virtual Environments
wpkg will fully support virtual environments, similar to how python supports them.

```
wpkg init-venv <PATH>
source <PATH>/bin/activate
```

This will set `$WPKG_HOME` as well as modify `PATH` to correctly point to only the
virtualenv location (removing the typical ones).
