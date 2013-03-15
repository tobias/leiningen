# GPG Primer

{{this is a work in progress - any comments/changes welcome. It likely
needs to be better linked to existing leiningen docs & clojars wiki
pages}}

This document aims to be "just enough" for setting up and using
[GPG](http://www.gnupg.org/) keys for signing artifacts with
[leiningen](http://leiningen.org) for publication to
[clojars](http://clojars.org/).  It is a work in progress - feel free
to provide pull requests that clarify/expand any part of it.

There are two versions of GPG available: v1.x and v2.x. For our
purposes, they are functionally equivalent. Package managers generally
install v2.x as `gpg2`, and v1.x as `gpg`. By default, Leiningen
expects the GPG command to be `gpg`. You're welcome to use any version
you like, but this primer will only cover installing v1.x, and has
only been tested under v1.x.

## What is it?

GPG (or Gnu Privacy Guard) is a set of tools for cryptographic key
creation/management and encryption/signing of data. If you are
unfamiliar with the concepts of public key cryptography, this
[Wikipedia entry](http://en.wikipedia.org/wiki/Public-key_cryptography)
serves as a good introduction.

An important concept to understand in public key cryptography is that
you are really dealing with two keys (a *keypair*): one public, the
other private (or secret). The public key can be freely shared, and is
used by yourself and others to encrypt data that only you, as the
holder of the private key, can decrypt. It can also be used to verify
the signature of a file, confirming that the file was signed by the
holder of the private key, and the contents of the file have not been
altered since it was signed. **You should guard your private key
closely, and share it with no one.**

## Installing GPG

### Linux

##### Debian based distros

Apt uses GPG v1, so it should already be installed. If not:

    apt-get install gnupg
    
#### Fedora based distros

Fedora and friends may have GPG v2 installed by default, but GPG v1 is
available via:
    
    yum install gnupg
    
### Mac

There are several options here, depending on which package manager you
have installed (if any):

1. via [homebrew](http://mxcl.github.com/homebrew/): `brew install gnupg`
2. via [macports](http://www.macports.org/): `port install gnupg`
3. via a [binary installer](https://www.gpgtools.org/installer/index.html)

### Windows

http://gpg4win.org/ {{untested, we should get a windows user to vet this doc}}

## Creating a keypair

Create a keypair with:

    gpg --gen-key

This will prompt you for details about the keypair to be generated,
and store the resulting keypair in `~/.gnupg/`.

The default key type (RSA and RSA) is fine for our purposes, as is the
default keysize (2048). {{what's a good recommendation for validity?
and rationale?}} 

You'll be prompted for a password to protect your private key - it's
important to use a strong one to help protect the integrity of your key.

Example:
```
$ gpg --gen-key
gpg (GnuPG) 1.4.11; Copyright (C) 2010 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Please select what kind of key you want:
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
Your selection? 
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (2048) 
Requested keysize is 2048 bits
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 1y
Key expires at Wed 12 Mar 2014 10:53:15 AM PDT
Is this correct? (y/N) y

You need a user ID to identify your key; the software constructs the user ID
from the Real Name, Comment and Email Address in this form:
    "Heinrich Heine (Der Dichter) <heinrichh@duesseldorf.de>"

Real name: Bob Bobson
Email address: bob@bobsons.net
Comment: 
You selected this USER-ID:
    "Bob Bobson <bob@bobsons.net>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? O
You need a Passphrase to protect your secret key.

We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.

Not enough random bytes available.  Please do some other work to give
the OS a chance to collect more entropy! (Need 284 more bytes)
..+++++
.+++++
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
.....+++++
..+++++
gpg: /home/bobson/.gnupg/trustdb.gpg: trustdb created
gpg: key DC0954B7 marked as ultimately trusted
public and secret key created and signed.

gpg: checking the trustdb
gpg: 3 marginal(s) needed, 1 complete(s) needed, PGP trust model
gpg: depth: 0  valid:   1  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 1u
gpg: next trustdb check due at 2014-03-12
pub   2048R/DC0954B7 2013-03-12 [expires: 2014-03-12]
      Key fingerprint = 15ED ECA5 17B2 9D79 D685  FA35 3108 230E DC09 54B7
uid                  Bob Bobson <bob@bobsons.net>
sub   2048R/6453D146 2013-03-12 [expires: 2014-03-12]
```

## Listing keys

GPG stores keys in a keystore located in `~/.gnupg/`. This keystore
holds your keypair(s), along with any other public keys you have used.

To list your private keys:

    gpg --list-secret-keys
    
This will produce arcane output similar to:

```
/home/bobson/.gnupg/secring.gpg
-------------------------------
sec   2048R/DC0954B7 2013-03-12 [expires: 2014-03-12]
uid                  Bob Bobson <bob@bobsons.net>
ssb   2048R/6453D146 2013-03-12
```

To list all of the public keys:

    gpg --list-keys
    
This will produce similar output, but will include any public keys you
have used (if you've never used GPG before, you should just see your
own public key):

```
/home/bobson/.gnupg/pubring.gpg
-------------------------------
pub   2048R/DC0954B7 2013-03-12 [expires: 2014-03-12]
uid                  Bob Bobson <bob@bobsons.net>
sub   2048R/6453D146 2013-03-12 [expires: 2014-03-12]

pub   2048R/958AE602 2011-05-13
uid                  Clojure/core (build.clojure.org Release Key version 2) <core@clojure.com>
sub   2048R/55DB4D58 2011-05-13
```

You can pass a filter string to either list command that will be
matched against any portion of the key's user-id:

```
$ gpg --list-keys bob
/home/bobson/.gnupg/pubring.gpg
-------------------------------
pub   2048R/DC0954B7 2013-03-12 [expires: 2014-03-12]
uid                  Bob Bobson <bob@bobsons.net>
sub   2048R/6453D146 2013-03-12 [expires: 2014-03-12]
```

## How Leiningen uses GPG

Leiningen uses gpg for two things: decrypting credential files, and
signing release artifacts. We'll focus on artifact singing here; for
information on credentials encrypting/decrypting, see
[DEPLOY](./DEPLOY.md).

### Signing a file

When you deploy a non-SNAPSHOT artifact to Clojars via the `deploy`
task, Leiningen will attempt to create GPG signatures of the jar and
pom files. It does so by shelling out to `gpg` and using your default
private key to sign each artifact. This will create a signature file
for each artifact named by appending `.asc` to the artifact name.

Both signatures are then uploaded to Clojars along with the
artifacts. In order for Clojars to verify the signatures, you'll need
to provide it with your *public* key (see below).

### Overriding the gpg defaults

By default, Leiningen will try to call GPG as `gpg`, which assumes
that `gpg` is in your path, and your GPG binary is actually called
`gpg`. If either of those are false, you can override the command
Leiningen uses for GPG by setting the `LEIN_GPG` environment variable.

Leiningen currently makes no effort to select a private key to use for
signing, and leaves that up to GPG. GPG by default will select the
first private key it finds (which will be the first key listed by `gpg
--list-secret-keys`). If you have multiple keys and want to sign with
one other than first, you'll need to set a default key for GPG. To do
so, edit `~/.gnupg/gpg.conf` and set the `default-key` option to the
id of the key you want to use. You can get the key id from the 'sec'
line in the secret key listing - the id follows the key length and
protocol specification.

For example, the following key is a **2048**-bit **R**SA key with an
id of **DC0954B7**:

    sec   2048R/DC0954B7 2013-03-12 [expires: 2014-03-12]

## Clojars 

Clojars requires that artifacts be signed and verified before being
promoted to the [releases] repository. In order to verify the
signature, it needs a copy of your *public* key. To view your public
key, use `gpg --export -a` giving it either the key id or the email
address associated with that key. Example:

```
$ gpg --export -a bob@bobsons.net
-----BEGIN PGP PUBLIC KEY BLOCK-----
Version: GnuPG v1.4.11 (GNU/Linux)

mQENBFE/a/UBCAChmZrZWFFgzzYrhOVx0EiUa3S+0kV6UryqkxPASbHZLml3RlJI
m36JD9dnioeuq1XyPdVUuqgR4JdSR6neZnWWI4AXaU2LjZRdGWLSGZmXB0/WB1gg
6qUp6VLzv8/pptHhZLHjAnD43iVQFyxJA+ahSv1e0LpBhpVVyasWAzYd65TwNgw9
z1lt2QUewW6V68HHdfYuqBAFSo425EjxusB8a5ITsSEV8cYDocIInN6OQHPuNdso
/zCKGSp3oNpeb186fEG0gqfTS9dSHmMY38A2hr3jQs+BsuUt7NRdrMoStKlQGQ1u
YZrAbWA+yGGaDeYhzvYt5bCCV8CLfcgow9TFABEBAAG0HEJvYiBCb2Jzb24gPGJv
YkBib2Jzb25zLm5ldD6JAT4EEwECACgFAlE/a/UCGwMFCQHhM4AGCwkIBwMCBhUI
AgkKCwQWAgMBAh4BAheAAAoJEDEIIw7cCVS3UdYH/1EGrcteDmoMjF0derhTi4ny
1Qfrl+UeMiQFAUrxEyN1aWXqdANgdHhY2w897x5rEVjPrWh4N4AbeCtvPgXXJ1DC
Gqqz8WSBF9VMhkU0QoRj0fIt7qRIUTLOl/9LzZQTlirWbhPMMuDgbDvyoz8rTuGu
/M1qHt+WyH1XdQIElXXCcxfmfVSbxQlC3uHDOD2euI5i1FxYe8PRqVNv8G/1StLF
qfOWkTEU4bmrLE4qiDM1g/9cipWGCVnCNQR3Gjiles4q37RHTlNRhW4EZCKSdze8
J3F2u+B1rw2KJIau/jo0EZ/R+g4gP6qImPgOfcJzEqhIopTXTNdODGImE0JL+qS5
AQ0EUT9r9QEIAMrIc51Kjeq0XPJvj/98UWBzRdMYswP7iTB82AqM77JW64vnsGQ9
oGQ007uTBjZTfq47j9iPtWttxeSICDnXQJndd/7yC/R9AyI6kLaT9fra1ZJHKkJi
2Luvh41oJOx2PMTyXQD9+qDkYGXVHopI7zR6ltY1Nbe6Y8byLKm2N4NWRBI/u7Yt
FZZcTLZR5RLyQ3edfAF2j6Ly17FOrf2toTM90265M/OVGbJ3X6k+EhsjKP0aw4wJ
F8gC+JGAOIdf9/VIXZPhTe2CzJSeH0Z4Le97wSQf4c0gvOPaW5Kbn+QzilFT4U37
zAbyqfdHM+79ni2gAKJZBXijdpoxGP1cMfcAEQEAAYkBJQQYAQIADwUCUT9r9QIb
DAUJAeEzgAAKCRAxCCMO3AlUt5ZLCACN/Nj2m/r3MuNDEnRVf2aicxZVCHuiFmEn
OHaqykhidhSqX85+xHn3u3ygEOst4X4lSToQNSFMj/1O7cy7jXTDAkxhC5jjAbZY
mf9Ikqvm2hTs/ZFWau/XMm3Ht4l4Kh6S1szcQ2gwV8xQr5Pl5/7VIs1BjbYA8Y4i
wn4BMETld5SqU5ap+KO/ZNrq+VC8hIynGzYdFSjHXcpG+QrjPkGKG6jae6WJJrx4
QPlFGYq7xOOd9fBrjDYlHsRMEzeF0SlT8kpgT6HzXVNlXKuloaSSeGh5ZRbGKNaw
Mc2Lhh+4KDbjozr08XXb1BpdyhEWPQJH1GwuH6XrFKnuRlVEvb+4
=EaPb
-----END PGP PUBLIC KEY BLOCK-----
```

Copy the entire output (including the BEGIN and END lines), and paste
it into the 'PGP public key' field of your Clojars profile.

### Web of trust

Verifying the signature is only part of the security story. Ideally,
you would also like to know that Bob's key actually belongs to
Bob. This requires face-to-face verification of Bob's identity and
key. But that type of verification isn't scalable for Clojars, since
we can't all meet each other face-to-face. 

Fortunately, GPG provides a solution for this - a [Web of Trust]. 

{{needs a description of the web of trust and signing parties:
http://cryptnet.net/fdp/crypto/keysigning_party/en/keysigning_party.html
And what about key servers? This section will likely be influenced by
the plan for the Clojure/West signing session}}

### deploy vs. scp

Currently, publishing signatures to Clojars only works if you are
using `lein deploy clojars`. If you are using `scp` to deploy, you can
copy signatures along with the artifacts, but they will be
ignored. {{is this true?}}
