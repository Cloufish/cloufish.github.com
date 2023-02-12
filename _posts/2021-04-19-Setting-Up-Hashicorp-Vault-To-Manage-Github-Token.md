---
title: Setting up Hashicorp Vault to manage our Github Token
date: 2021-04-19 06:46:00 +0100
categories: [DevSecOps, Vault]
tags: [vault, authentication, key management, api,devsecops]
lang: en
---

> UPDATE: This blog post is about managing secrets to authenticate to GitHub via HTTPS connection. But you can also authenticate via SSH connection and generate your own private key as described [here](https://stackoverflow.com/questions/6565357/git-push-requires-username-and-password). This solution may be more comfortable, because you're not even prompted for **any** credentials.

## Intro
If you're working with GitHub/git you may have noticed that you got an e-mail which informs you about a **Deprecation of Username:Password authentication via git in favor of authentication of Username:Token by the end of *12th August*


And you've probably got the links below to generate new API Github Token as well as why they want you to do that:
- https://github.blog/2020-12-15-token-authentication-requirements-for-git-operations/

- https://docs.github.com/en/github/authenticating-to-github/creating-a-personal-access-token

It's a trivial thing to do, so I'll skip the process of generating the Token.

Some of you might be thinking on how to exactly store this Token securely?

## How tokens/secrets should be kept **secret**:
### Level 0 - Hardcoded:
Definitely they shouldn't be hard-coded into our scripts, application code - This shouldn't even take place.
### Level 1 Putted in config file:

Putting them into a config file - like when assigning them into an Environment Variable e.g. ```GITHUB_TOKEN=gkw4k4uk5jkwrkf3k0j60fk23gksmbs```is also a bad idea, because a bad actor might still read these token when accessed to our machine, or even worse - we could accidentally push this config file into a public repository, because we didn't include this file into ```.gitignore```.

### Level 2 - Encrypted keys in config file/code:

If we need to push these keys into our repository, then they should definitely be encrypted

We can encrypt these keys with OpenSSL

But the process of decrypting these keys might be troublesome and tiring to an ordinary developer.

And also it puts the question - **Where to store the encryption key?**

- **If you wanted simplicity**, then you probably put these **encryption keys into a one file**. This though poses a risk of over-privileged access, because a person that wants to access only one key now has the access to all of them
- If you put these keys into different files, now you'd have bigger complexity and many of people would get frustrated of doing such a 'simple security thing'.

### Level 3 - Moving the secrets to dedicated server secret manager
- But creating your own crypto system has a lot of risks of not implementing it correctly. Just like the saying goes ```You shouldn't write your own crypto``` it can also apply to ```You shouldn't write your own crypto-server```
- You would probably use Cloud Service providers to manage your keys such as **Azure Key Vault** **AWS KMS**
- This will provide a high liability on who accessed these secrets and functionality
### Level 4 - Hashicorp Vault (Temporary Credentials)
- If the secret is stolen/disclosed/leaked then after a while It won't be valid
- It grants you a credentials/secrets and then after a while it revokes these credentials
## Development vs Production environment:

When thinking about managing key/tokens/secrets we should take into consideration on where they will appear.

In our case — We need to store Github Token ID in our Development Environment, so We could probably stay save at level 1 (If we didn't push the config file accidentally)

## Why still proceed further 'above' level 1?
- To learn. - That's basically it.

## #1 Installing Vault
- For this I recommend you head into [Hashicorp Vault Installation Docs](https://www.vaultproject.io/downloads).
I'm on arch linux so a command ```sudo pacman -Syu vault``` is enough for me to install vault

## #2 Turning on Vault server
- With command ``` vault server -dev ```
After executing this command, we'll see an important information for further configuration
![information](https://imgur.com/DUq7Q01.png)

## #3 Assigning $VAULT_TOKEN and $VAULT_ADDR
We need to overwrite these tokens with commands:
```export VAULT_ADDR='http://127.0.0.1:8200'``` and also your token with ```export VAULT_TOKEN="<ROOT_TOKEN_HERE>"```  
Hashicorp ```vault``` command uses its own API, If we wouldn't assign the $VAULT_ADDR, then every API request would be done with **https** protocol. Every request like that won't work, because we only run localhost environment with **http** protocol
![error](https://imgur.com/tZKUoiB.png)
## #4 Logging in to vault
With command ```vault login```

We could accomplish the same thing without the prompt with the help of piping our $ROOT_TOKEN
```echo $VAULT_TOKEN | vault login -```

## Setting up Username:password based authentication
Logging in with root token can be troublesome, especially if we don't need this many permissions in our daily usage. So let's set up different authentication method called *userpass* with these steps:

### 1. Setting up $VAULT_ADDR and $VAULT_TOKEN as previously
### 2. Login with ```vault login```
### 3. Enabling userpass
- ```vault auth enable userpass```
    > *TIP* - You can always execute command ```vault path-help auth/userpass``` to get further info on how to use this auth method
    This is so awesome, because It also tells us that we can implement Multi-Factor-Implementation - that's crazy!
    Also it tells us how to reset a password, update their policies and which users can authenticate
### 4. Creating user
- ```vault write auth/userpass/users/<YOUR_CHOSEN_USERNAME> password=<YOUR_CHOSEN_PASSWORD>``` To create an user
    > We can confirm if we managed to add a new user with ```vault  list/userpass/users``` command
1. We could login now with: ```vault login -method=userpass --username=<YOUR_CHOSEN_USERNAME>``` command, but hang on! Because we won't be able to access our $GITHUB (TOKEN), because we don't have as a new created user permission to read them! Firstly we should create this policy

### 5. Writing a policy to access our secrets in secret/ vault

The command syntax to write a new policy is:
- ```vault policy write [name_of_new_policy] [policy_file.hcl] ```
So it requires a [policy_file] which we apparently do not have. Let's write it then.
```github.hcl```:
```
path "secret/*" {
    capabilities = ["read", "list"]
}
```
That's seriously it for our use case! :D. We allow read and list operations. Let's write it with command:
- ```vault policy write github github.hcl ```

> **If you encounter any parsing issues this the command above, use:**
```json
vault policy write github - << EOF
path "secret/*" {
    capabilities = ["read", "list"]
}
EOF
```
And now let's assign this policy to our account:
```vault write auth/userpass/users/<YOUR_CHOSEN_USERNAME> password=<YOUR_CHOSEN_PASSWORD> policies=github```

Now, I know you want to login and test it out!, But let's add those secrets first! (Because as the unprivileged user we won't have this luxury! :P)
## #5 Putting your Github key to Hashicorp Vault
with:
```vault kv put secret/email key=<YOUR_EMAIL_HERE>```

Also, I want to add secret called **email**. Now, many of you might argue if it's really a secret... I'd be using it in shell scripts, so I think I'll recognize it as a secret

```vault kv put secret/email key=<YOUR_TOKEN_HERE>```
## #6 Logging in and testing these policies!
Now lets login with:
```vault login -method=userpass --username=<YOUR_CHOSEN_USERNAME>```

Now - **the important part** is to overwrite our VAULT_TOKEN variable with the user token.

![user_token](https://imgur.com/tYpNEf4.png)

We can see the big **WARNING!** there. Essentially if we won't switch then we would still be interacting with the Hashicorp's API with our root token.

If you've done everything correctly then you've got the Hashicorp Vault set-up!

## #7 Migrating Dev Environment to 'Production'
Now that we've seen the process of configuration, let's begin to work with non-dev Hashicorp's vault environment, there are several reasons to do that:
- The Dev environment runs in 'unsealed' state by default - This essentially means that all the secrets are decrypted at
the start of the vault.
- Dev environment also runs with storage in-memory. Meaning that it offers no persistence to the secrets We're storing. All data is lost when Vault or the machine on which it is running is restarted.

The whole process is explained in Hashicorp Vault's Docs about [Deploying](https://learn.hashicorp.com/tutorials/vault/getting-started-deploy)
### 1. Writing config.hcl

I've prepared one with following contents:
```json
storage "file" {
  path    = "/mnt/vault/data"
}

listener "tcp" {
  address     = "127.0.0.1:8200"
  tls_disable = "true"
}


api_addr = "http://127.0.0.1:8200"
ui = true

```
I'm using file system storage backend with
```json
storage "file" {
  path    = "/mnt/vault/data"
}
```
We need to also create this path with ```sudo mkdir /mnt/vault/data".

And we can start our server with the command:
```vault server -config=config.hcl```

### 2. Vault init:

After starting vault server we can access the website with basic 'initialization'
![init](https://imgur.com/1qHRBzF.png)

It tells us to specify the number of key shares - These tokens will be used to the process of *unsealing vault* - each key should be owned by each person administrating the vault. Because It's a local setup I'll set each of the form input to '1'

After that:
![init-success](https://imgur.com/U210xK3.png)

I recommend downloading the file with keys, moving this file **outside of your home folder**, changing the owner of this file to root user with
```chown root <vault_key_file>```
 and setting up permission to read-only with
```chmod 400 <vault_key_file>```

There's also root key to login
### 'Migrating'... Yeah...
I didn't expect that to be honest, but we have to repeat the whole process of creating a user, adding policy again. To make things easier for you and myself I've created this simple bash script:

```bash
#!/bin/bash

export VAULT_ADDR="http://127.0.0.1:8200"
export VAULT_TOKEN=""
YOUR_CHOSEN_USERNAME=""
YOUR_CHOSEN_PASSWORD="" # ENTER YOUR PASSWORD HERE, OR ENVIRONMENT VARIABLE WHERE THE PASSWORD IS LOCATED
YOUR_EMAIL_HERE=""
YOUR_GITHUB_TOKEN_HERE=""


echo $VAULT_TOKEN | vault login -

vault auth enable userpass

vault policy write github - << EOF
path "secret/*" {
    capabilities = ["read", "list", "create", "update"]
}
EOF



vault write auth/userpass/users/"${YOUR_CHOSEN_USERNAME}" password="${YOUR_CHOSEN_PASSWORD}" policies=github

vault secrets enable -path=secret/ kv

vault kv put secret/github/email key="${YOUR_EMAIL_HERE}"
echo $VAULT_TOKEN
vault kv put secret/github/GITHUB_TOKEN key="${YOUR_GITHUB_TOKEN_HERE}"

unset VAULT_TOKEN

vault login -method=userpass username="${YOUR_CHOSEN_USERNAME}"
```
After the execution of the script, ```export VAULT_TOKEN=<YOUR_USERNAME_TOKEN>``` and test it.

After testing remove the script completely (don't forget the trash directory) or delete the hard-coded credentials

## Integrating it with a bash alias:

One thing is missing from the Hashicorp's documentation and that is - lack of setting up vault as systemd service. Fortunately I stumbled across this blog post:

https://blog.vivekv.com/hashicorp-vault-systemd-startup-script.html

We don't need to do everything described in this blog. There are little to no steps:

1. Open the file ```/usr/lib/systemd/system/vault.service```
and see the contents of it:

```
[Unit]
Description=Vault server
Requires=basic.target network.target
After=basic.target network.target

[Service]
User=vault
Group=vault
PrivateTmp=yes
ProtectSystem=full
ProtectHome=read-only
CapabilityBoundingSet=CAP_IPC_LOCK
Environment=GOMAXPROCS=2
ExecStart=/bin/vault server -config=/etc/vault.hcl
KillSignal=SIGINT
TimeoutStopSec=30s
Restart=on-failure
StartLimitInterval=60s
StartLimitBurst=3

[Install]
WantedBy=multi-user.target
```

We want to replace the existing ```/etc/vault.hcl``` file with our config. with

```sudo cp <PATH_TO_OUR_CONFIG> /etc/vault.hcl```


2. Execute
   ```systemctl daemon-reload && systemctl start vault && systemctl enable vault```

   then check if it has successfully started with
    ```systemctl status vault```. If it didn't, then check the logs with:
    ```sudo journalctl -u vault.service```

Also let's add automatically export the ```$VAULT_ADDR``` variable. We could do that in ```.bashrc```, ```.bash_profile``` or just like the author of the post did, place a bash script here ->```/etc/profile.d/vault.sh``` with the contents of:
```bash
#!/bin/bash
export VAULT_ADDR='http://127.0.0.1:8200'

```

Now let's begin with automating the process of getting these keys, unfortunately, it's hard to implement getting the keys automatically with ```git push``` command. We may get the idea of doing that later... For now though, let's create an **alias** for the set of vault commands.

```bash
function gitgot(){

	vault login username=<YOUR_USERNAME_HERE>
	vault kv get -field=key secret/github/email
	vault kv get -field=key secret/github/GITHUB_TOKEN
}
```
Put this function inside a ```.bash_profile``` file or ```.zprofile``` file

Now when you start a new terminal, executing command ```gitgot``` should ask you for username and password, and if credentials were correct both email and GITHUB_TOKEN would be printed out.

We can also set caching of these credentials when doing ```git push``` to be around 4hours with command:

```bash
git config --global credential.helper 'cache --timeout 14400'
```

## Caveats:

- We've used a default secrets path called ```secret/github```. I've done that because I don't see (yet!) another use-case of Hashicorp Vault on my daily basis workflow. However, If I had to manage more secrets and more API keys I would probably diverse these tokens to different profiles.

**That's it!** It was seriously a long struggle for me, I personally encountered many issues with setting this up, and though It may not be perfect having vault implemented feels so satisfying! I hope you've also learned something and because that scenario is meant to run only locally maybe you'll do the setup yourself :) Cheers.
