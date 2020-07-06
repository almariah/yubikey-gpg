# GPG

If you want to delete everything after creating or playing with keys

```bash
gpg --delete-secret-key "Abdullah Almariah"
gpg --delete-key "Abdullah Almariah"
```

To list all keys:

```
gpg -k
```

## Steps

* Configure `GnuPG`:

```bash
mkdir ~/.gnupg

cat > ~/.gnupg/gpg.conf << EOF
no-emit-version
no-comments
keyid-format 0xlong
with-fingerprint
use-agent
personal-cipher-preferences AES256 AES192 AES CAST5
personal-digest-preferences SHA512 SHA384 SHA256 SHA224
cert-digest-algo SHA512
default-preference-list SHA512 SHA384 SHA256 SHA224 AES256 AES192 AES CAST5 ZLIB BZIP2 ZIP Uncompressed
EOF
```

* Generate Master Key with these spec:
    - `RSA (sign only)`
    - `4096` bits
    - for `5y`
    - `Abdullah Almariah`
    - `abdullahalmariah@gmail.com`

```bash
gpg --full-generate-key
```

After running the previous command, we can get the key ID (example: `0xD60BAB29C43A7D86`):

```
...
Note that this key cannot be used for encryption.  You may want to use
the command "--edit-key" to generate a subkey for this purpose.
pub   rsa4096/0xD60BAB29C43A7D86 2018-10-06 [SC] [expires: 2020-10-05]
...
```

* Generate sub-keys for signing:

```bash
gpg --expert --edit-key 0xD60BAB29C43A7D86
```

Then write `addkey`, and select `RSA (sign only)` with `4096` size and for `3y` finally exit `q`

* Generate sub-keys for encryption:

```bash
gpg --expert --edit-key 0xD60BAB29C43A7D86
```

Then write `addkey`, and select `RSA (encrypt only)` with `4096` size and for `3y` finally exit `q`

* Generate sub-keys for authentication:

```bash
gpg --expert --edit-key 0xD60BAB29C43A7D86
```

Then write `addkey`, and select `RSA (set your own capabilities)` with `4096` size and for `3y` finally exit `q`. You need to toggle toggle `s`, `e`, `a` then `q`.

* Generating a revocation certificate to be used inf future if you lost access for the keys. Make sure that `~/gpg/` is empty:

```bash
mkdir ~/gpg

gpg --gen-revoke 0xD60BAB29C43A7D86 > ~/gpg/revoke.asc
```

Then choose `Key has been compromised`.

* Backup keys:

```bash
gpg -a --output ~/gpg/0xD60BAB29C43A7D86.master.asc --export-secret-key 0xD60BAB29C43A7D86
gpg -a --output ~/gpg/0xD60BAB29C43A7D86.subkeys.asc --export-secret-subkey 0xD60BAB29C43A7D86
```

* Backup the following files to 2 usb sticks and keep them in safe place.
    - `~/gpg/0xD60BAB29C43A7D86.master.asc`
    - `~/gpg/0xD60BAB29C43A7D86.subkeys.asc`
    - `~/gpg/revoke.asc`
    - `~/.gnupg/gpg.conf`

* Trust the keys by the following command and then type `trust` and choose `I trust ultimately`

```bash
gpg --edit-key 0xD60BAB29C43A7D86
```

* Prepare the yubikey

```bash
 gpg --card-edit
```

then type `admin`, then `passwd`. Now change the password if it is the first time by selecting `change PIN`. If the first time the old PIN is `123456`. Then change admin PIN (`change Admin PIN`). The old if it is the first time, the old PIN is `12345678`

Then exit `passwd` menu (`q`). Then type `url`and set to `https://keybase.io/test/key.asc`.

Set the name by typing `name` and set `Almariah` and `Abdullah`.

Set the login by typing `login` and then set to `almariah`.

Set the sex by typing `sex` and then set to `M`. Also set the language by typing `lang` and set to `en`.

Finally quit by `q`.

* Moving keys into yubikey:

```bash
gpg --edit-key 0xD60BAB29C43A7D86
```

then type `toggle`. Now select key 1 by typing `key 1` and then type `keytocard`.

Select the type as `Signature key`.

Now un-toggle key 1 by typing `key 1` and then toggle key 2 by typing `key 2`. Repeat the previous steps but with `Encryption key` for key 2. Further repeat again for key 3 with `Authentication key`. Finally type `save`

* Enabling Touch-to-Sign/Decrypt (for more security if something happened in background):

```bash
# it could be better to try it on mac after installing yubikey-manager
ykman openpgp touch enc on
ykman openpgp touch sig on 
```

* Share the public key by exporting the public key using

```
gpg --export -a --output ~/gpg/0xD60BAB29C43A7D86.pub.asc 0xD60BAB29C43A7D86
```

* Adding a GPG key to Keybase.io

```bash
sudo yum install https://prerelease.keybase.io/keybase_amd64.rpm

# drop old key if needed
# keybase pgp drop [KEYID]

# keybase login ...

# Select your public key
keybase pgp select
```

## Configure gpg-agent

First install GPG suit on macOS (disable Mail.app plugin since it costs memory). Then configure the gpg-agent (`$HOME/.gnupg/gpg-agent.conf`) as follow:

```
default-cache-ttl 600
max-cache-ttl 7200
enable-ssh-support
```

To use `gpg-agent` with `SSH` we need to set `SSH_AUTH_SOCK` as follow:

```bash
export SSH_AUTH_SOCK=$(gpgconf --list-dirs agent-ssh-socket)
```

You can export to bash (`$HOME/.bashrc`) or Zsh `$HOME/.zshrc` config.

Now you can import the GPG public key by:

```bash
gpg --import 0xD60BAB29C43A7D86.pub.asc

# or
curl https://keybase.io/almariah/pgp_keys.asc | gpg --import
```

Then you can plug in the yubikey and check its status:

```bash
gpg --card-status
```

Then trust the key (optional):

```bash
# to list keys and then find the pub key ID
# gpg -k

gpg --edit-key SOME_ID... 
```

Then type `trust` to trust the key.

To check SSH agent if the SSH public key has been added:

```
ssh-add -L
```


Not sure

```bash
export GPG_TTY="$(tty)"
export SSH_AUTH_SOCK=$(gpgconf --list-dirs agent-ssh-socket)
gpgconf --launch gpg-agent
```


## Encrypt message

## Github

You can use YubiKey to sign GitHub commits and tags. To configure a signing key:

```bash
git config --global user.signingkey SOME_KEY_ID...
```

then commit with `-S` option.