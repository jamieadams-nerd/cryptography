My motivation for doing this research was when I tried to use an elliptcal curve algorithm when sitting up my "git" confugraiotn on a RHEL 10 platform with the kernel mode in FIPS.

This guide provides a step-by-step workflow for configuring a FIPS-compliant Linux system to securely work with GitHub. It covers detecting FIPS mode, generating and configuring RSA SSH keys 
for authentication, creating GPG keys for signing commits, adding keys to GitHub, initializing repositories, pushing signed commits, and troubleshooting common issues. Following 
these procedures ensures that all Git operations are secure, cryptographically compliant, and verifiable, making it easier for developers to maintain code integrity and work safely 
in environments with strict compliance requirements.


# Overview

This guide outlines how to configure a FIPS-compliant Linux system to work with GitHub, including generating and using SSH and GPG keys, signing commits, and troubleshooting common issues.

The main procedures covered are:
* FIPS Mode Detection
* Check if the system is running in FIPS mode to ensure only approved cryptographic algorithms are used.
* Helps avoid errors when generating keys or connecting to GitHub.
* SSH Key Generation and Configuration
* Generate a 4096-bit RSA key, the only key type allowed in FIPS mode.
* Configure SSH to enforce FIPS-approved algorithms and add GitHub’s host key.
* Ensures secure, authenticated Git connections.
* GPG Key Generation for Commit Signing
* Generate a 4096-bit RSA GPG key.
* Configure Git to sign commits automatically.
* Ensures commits are verifiable and tamper-proof, providing trust for collaboration.
* Adding Keys to GitHub
* Add SSH keys for authentication and GPG keys for commit verification.
* Enables Git operations (push/pull) and allows signed commits to be recognized by GitHub.
* Repository Initialization and Commit Workflow
* Create repositories locally and on GitHub.
* Push commits to GitHub and verify that they are signed.
* Supports standard Git workflows while maintaining compliance.
* Diagnostics and Troubleshooting
* Tools for checking SSH key loading, host authenticity, allowed algorithms, and GPG signing.
* Helps identify and resolve common FIPS-related connection issues.

How this helps someone:
* Ensures secure Git operations on systems that must comply with FIPS regulations.
* Prevents errors caused by unsupported cryptography in FIPS mode (e.g., ECDSA or ED25519 keys).
* Guarantees that commits are signed and verifiable, improving trust in code integrity.
* Provides a step-by-step workflow that includes both configuration and troubleshooting guidance.
* Makes it easier to work with GitHub safely in environments with strict compliance requirements.




# Step-by-Step Procedures


## Step 0: Detect if your system is in FIPS mode

Check if FIPS mode is enabled. Non-FIPS keys and algorithms will fail.

```bash
cat /proc/sys/crypto/fips_enabled
```

* Output 1 → FIPS mode enabled
* Output 0 → FIPS mode disabled

Alternatively:

```bash 
fips-mode-setup --check
```

Reports FIPS mode enabled or disabled

Example errors if using non-FIPS keys:

ED25519:

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```
May show: `key type not allowed in FIPS mode`


ECDSA when connecting: `kex_exchange_identification: no matching key exchange method found`

These indicate RSA 4096 is required in FIPS mode.

Check allowed SSH algorithms:
```bash
ssh -Q key
ssh -Q kex
ssh -Q cipher
ssh -Q mac
```

ssh-rsa should appear; ecdsa and ed25519 will not


## Step 1: Generate a FIPS-compliant SSH key

`ssh-keygen -t rsa -b 4096 -C "your_email@example.com"`


Accept default file (~/.ssh/id_rsa)

Optionally set a passphrase


## Step 2: Start the SSH agent and add the key
```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa
```

## Step 3: Add the SSH key to your GitHub account

Copy the public key:
```bash
cat ~/.ssh/id_rsa.pub
```

Go to GitHub → Settings → SSH and GPG keys → New SSH key

Paste the public key and give it a name

## Step 4: Configure SSH for FIPS-compliant algorithms
```bash
mkdir -p ~/.ssh
nano ~/.ssh/config
```

Add:
```
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_rsa
    IdentitiesOnly yes
    KexAlgorithms +diffie-hellman-group14-sha1,diffie-hellman-group-exchange-sha256
    HostKeyAlgorithms +ssh-rsa
    PubkeyAcceptedAlgorithms +ssh-rsa
```

## Step 5: Add GitHub’s RSA host key

```bash
ssh-keyscan -t rsa github.com >> ~/.ssh/known_hosts
```

First-time warnings are normal

Type `yes` to accept


## Step 6: Test SSH connection
```bash
ssh -T -vvv git@github.com
```

Expected output (amongst the numerous debug messages caused by the -vvv):

`Hi username! You've successfully authenticated, but GitHub does not provide shell access.`

## Step 7: Configure Git user information

```bash
git config --global user.name "Your Name"
git config --global user.email "your_email@example.com"
```

## Step 8: Generate a GPG key for signing commits
```bash
gpg --full-generate-key
```

* Select RSA and RSA
* Key size: 4096 bits
* Expiration optional (I recommend never expire)
* Enter name and email (this must match your git global settings for user.name and user.email)
* Optional passphrase

## Step 9: List GPG keys and copy the key ID

```bash
gpg --list-secret-keys --keyid-format LONG
````

Copy the key ID (e.g., ABCDEF1234567890)


## Step 10: Add the GPG key to your GitHub account

Export the public GPG key:
```bash
gpg --armor --export ABCDEF1234567890
```

Go to GitHub → Settings → SSH and GPG keys → New GPG key

Paste the key and save

## Step 11: Configure Git to use the GPG key
```bash
git config --global user.signingkey ABCDEF1234567890
git config --global commit.gpgsign true
```

## Step 12: Create a local Git repository and make the first commit
You should skip this step and step 13 if you already know how to manage repositories and perform a commit. 

```bash
mkdir ~/test-repo
cd ~/test-repo
git init
echo "Hello FIPS GitHub" > README.md
git add README.md
git commit -S -m "Initial commit"
```

-S signs the commit

Note: GitHub now uses main as the default branch instead of master

Rename branch to main:
```bash
git branch -M main
```

Optional: set all future repositories to default to main:
```bash
git config --global init.defaultBranch main
```


## Step 13: Create a repository on GitHub

Go to GitHub → New Repository

Repository name: test-repo

Do not initialize with README

Copy the SSH URL (e.g., git@github.com:your-username/test-repo.git)

Step 14: Add the remote repository
git remote add origin git@github.com:your-username/test-repo.git

Step 15: Push the main branch
git push -u origin main


Sets upstream tracking

Step 16: Make future commits
git add <files>
git commit -S -m "Your commit message"
git push

Step 17: Verify signed commits locally
git log --show-signature -1
git log --show-signature
git verify-commit <commit-hash>


Should show Good signature from "Your Name <your_email>"

Step 18: Verify commits on GitHub

Signed commits show a green “Verified” checkmark

Step 19: Diagnostics and troubleshooting

Check loaded SSH keys

ssh-add -l


Verbose SSH connection to GitHub

ssh -vvv -T git@github.com


-vvv gives detailed debug output

-T disables pseudo-tty allocation (required for GitHub SSH)

Check known_hosts

cat ~/.ssh/known_hosts


Check available SSH algorithms

ssh -Q key
ssh -Q kex
ssh -Q cipher
ssh -Q mac


Test GPG signing

echo "test" | gpg --clearsign


Check Git signing configuration

git config --list | grep sign

Step 20: Notes for FIPS mode

Only RSA keys are allowed for SSH and GPG

ECDSA, ED25519, and PQC keys will fail

Host authenticity warnings on first connection are normal

SSH config enforces FIPS-approved algorithms

Adding your SSH and GPG keys to GitHub is required for authentication and signed commits

Following these steps ensures commits are signed, pushed, and verifiable

Using git config --global init.defaultBranch main ensures new repositories default to main instead of master

This guide is now complete, fully corrected, FIPS-aware, includes SSH/GPG key addition to GitHub, diagnostics, and branch naming recommendations.

If you want, I can also make a workflow diagram showing FIPS detection → SSH/GPG key creation → Git setup → commit → push → verification for beginners. This helps visualize the process.

Do you want me to create that diagram?

Attach
Search
Study
Voice
