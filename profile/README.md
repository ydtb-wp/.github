## Hi there 👋

**Tracking Private Wordpress Plugins:**

This GitHub organization tracks private wordpress plugin updates for my agency and allows them to be used with Composer as a repository source. 

# Composer/SatisPress - Github Org Mirror 
[Mirror Repo](https://github.com/ydtb-wp/ydtb-wp.github.io)

What this does. I have multiple wordpress sites that use satispress to generate a separate composer repository for each group of paid private plugins. This repository allows me to aggragate all of those repositories into a single composer repostiory that can be used to install all of the plugins from a single location.

# How it works
That repo is designed to be used as the root repository for a GitHub organization. It can be a free org and is designed to maintain the secrets required to run the repository. (I.E., the VaultPass {encrypting and decrypting SatisPress un/pw }, and the GitHub PAT token used to push updates to the private repos). In this example, the org is called `ydtb-wp`, and the repository is called `ydtb-wp.github.io.` This is so that the composer repository can be accessed at `https://ydtb-wp.github.io` and the composer packages.json can be accessed at `https://ydtb-wp.github.io/packages.json`. If you are setting this up for your org, you must replace `ydtb-wp` with your org name.

![image](https://github.com/user-attachments/assets/a929274a-9a4a-41f0-8ed1-469f8f26a476)



## Updating composer

We can now add the following to our composer.json file for the site we are working on:
```json
    "repositories": [
        {
            "type": "composer",
            "url": "https://wpackagist.org",
            "only": [
                "wpackagist-plugin/*",
                "wpackagist-theme/*"
            ]
        },
        {
            "type": "composer",
            "url": "https://ydtb-wp.github.io/"
        }
    ]
```
I am using Trellis to deploy my wordpress sites, so I have added the GitHub credentials to the site vault. [Composer HTTP Basic Authentication](https://roots.io/trellis/docs/composer-http-basic-authentication/)

## ENV Variables
If you are trying to replicate this for your own org, you can set up your own org and clone the ydtb-wp.github.io locally, change the name, and then push it into your own org. 
You will need to create a `.env` file at the root of the repository. This file should contain the following variables:

```
VAULT_PASS=your-vault-pass #used to encrypt/decrypt the satispress username/password
PAT=your-github-pat #used to push updates to the private repositories
ORG=ydtb-wp #the name of the github org that this repository is in
REPO=ydtb-wp.github.io #the name of the repository that this is in
```

Under the github repository settings, you will need to add the following secrets:

```
VAULT_PASS=your-vault-pass #used to encrypt/decrypt the satispress username/password
PAT=your-github-pat #used to push updates to the private repositories
```
Note: the org and the repo are not needed as secrets, as they are already available to the actions.

## Executables
in the `.\bin` folder are several scripts used to set up, update, or check for plugin updates. 
most scripts are designed to be run from your local machine interactively, as they will prompt you for the required information. The updatePackages.ts script can be run locally but is also designed to be run as a GitHub action. It's set to run every 8 hours, but you can change that to whatever you want in the `.github/workflows/check-for-updates.yml` file.

* **addSource.ts** - This script will add a new satispress website/repository to the `./data/sources.json` file. It will prompt you for the necessary information and then add the new source to the file. The usernames and passwords are encrypted, not stored in plain text.
* **chooseRepos.ts** - This will fetch the list of plugins that all the satispress repositories are tracking and then prompt you to choose which ones you want to track in this repository. It will then update the `./data/packages.json` file with the selected plugins.
* **encryptSource.ts** - This will encrypt the username and password for a satispress repository. It will use the `VAULT_PASS` environment variable to encrypt the username and password and then update the `./data/sources.json` file with the encrypted values.
* **decryptSource.ts** - This will decrypt the username and password for a satispress repository. It will use the `VAULT_PASS` environment variable to decrypt the username and password and then update the `./data/sources.json` file with the decrypted values.
* **updatePackages.ts** - This is the main function that checks for new plugins and stores those tags as new versions in repos in the org. It then updates the `./data/packages.json` file with the new versions of the plugins. After all plugins are updated then it will read the ./data/packages.json and generate a new `packages.json`, which is available at `https://ydtb-wp.github.io/packages.json`

This can be run locally, but it is intended to run as a GitHub action. 

## Github Actions

### build-deploy.yml
This action takes the information from the `./data/packages.json` file and generates the composer `packages.json` file downloaded and used by composer to install the plugins.

### check-for-updates.yml
This action is run every 8 hours or whatever interval you specify in the workflow cron. It will check the multiple satispress repositories for any updates to the plugins that are tracked with the respective satispress repositories. If there are any, it will download the zip file for that tag and unzip it into the repo that tracks that specific plugin. If a repo does not exist, one will be created in the org. After it successfully merges the unzipped plugin into the repository, the repo is tagged with the tag version, and the `./data/packages.json` is updated to show that the new version is available. After all the updates have been run, and if there were updates that happened, then the information from `./data/packages.json` is used to generate a new `packages.json` file for Composer to use. 

# Work in Progress
This is a work in progress, and I am still working on the documentation and the scripts. I will update you on this as I make progress. If you have any questions, feel free to open an issue, and I will do my best to help you.


