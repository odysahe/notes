# My Exprience about UBUNTU
In this section all Exprience for Ubuntu 22.04
## Error GPG When run `sudo apt update`
Like this :
```
Reading package lists... Done
W: https://dl.yarnpkg.com/debian/dists/stable/InRelease: Key is stored in legacy trusted.gpg keyring (/etc/apt/trusted.gpg), see the DEPRECATION section in apt-key(8) for details.
```
The solution is:
`sudo cp /etc/apt/trusted.gpg /etc/apt/trusted.gpg.d/trusted.gpg` , i found the solution from this [link](https://youtu.be/8s0fTh1TD8k)
## Error `NO_PUBKEY` Public Key Not Available
If error like :
```
Reading package lists... Done
W: GPG error: http://download.opensuse.org/repositories/home:/manuelschneid3r/xUbuntu_22.04  InRelease:
The following signatures couldn't be verified because the public key is not available: NO_PUBKEY 1122EB22E192A122
E: The repository 'http://download.opensuse.org/repositories/home:/manuelschneid3r/xUbuntu_22.04  InRelease' is not signed.
N: Updating from such a repository can't be done securely, and is therefore disabled by default.
N: See apt-secure(8) manpage for repository creation and user configuration details.
```
The solution is : `sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 1122EB22E192A122`, i found the solution from this [Link](https://www.youtube.com/watch?v=M_h46Rv6X5Q)
> [!WARNING]
> the `--recv-keys 1122EB22E192A122` section adjust to your own key in the `NO_PUBKEY` section
