# Handout manual installation SUSECON 2024

This is a handout for the Manual installation guide for [Trento](https://github.com/trento-project/docs/blob/main/guides/manual-installation.md). 
The main reason for this is to provide references and helpers to make the installation easier to follow.

## Add trento-web and trento-wanda repository
Currently, the trento rpm's are in the release process and will be soon available for SLE15 SP5. As this process is not finished, add the alternative source repository.
```
sudo zypper ar -f https://download.opensuse.org/repositories/devel:/sap:/trento:/factory/SLE_15_SP5/ devel:sap:trento:factory
```
After adding the Open Build Service libary with trento's rpm, it is a must to refresh the repository:
```
sudo zypper ref
```

```As next step the user is asked about the new Repository:

New repository or package signing key received:

  Repository:       devel:sap:trento:factory
  Key Fingerprint:  6218 F543 A429 F63F 1401 CD8F 809F 4A42 31FF 744A
  Key Name:         devel:sap OBS Project <devel:sap@build.opensuse.org>
  Key Algorithm:    RSA 2048
  Key Created:      Sat Jan 28 19:38:18 2023
  Key Expires:      Mon Apr  7 20:38:18 2025
  Rpm Name:         gpg-pubkey-31ff744a-63d56b9a



    Note: Signing data enables the recipient to verify that no modifications occurred after the data
    were signed. Accepting data with no, wrong or unknown signature can lead to a corrupted system
    and in extreme cases even to a system compromise.

    Note: A GPG pubkey is clearly identified by its fingerprint. Do not rely on the key's name. If
    you are not sure whether the presented key is authentic, ask the repository provider or check
    their web site. Many providers maintain a web page showing the fingerprints of the GPG keys they
    are using.

Do you want to reject the key, trust temporarily, or trust always? [r/t/a/?] (r):
```

Press t to trust and install Trento rpm's

```zypper install trento-web trento-wanda```

## RPM configuration env helper
The next step requires to configure trento-web and trento-wanda.
To create a file with secrets in the current directory execute: 

```
{ 
  echo "SECRET_KEY_BASE=$(openssl rand -out /dev/stdout 48 | base64)"; 
  echo "ACCESS_TOKEN_ENC_SECRET=$(openssl rand -out /dev/stdout 48 | base64)"; 
  echo "REFRESH_TOKEN_ENC_SECRET=$(openssl rand -out /dev/stdout 48 | base64)"; 
} > secrets.txt && \
file_path="$(pwd)/secrets.txt" && \
echo -e "Trento web and wanda keys have been generated and saved to \033[0;31m$file_path\033[0m"

```

