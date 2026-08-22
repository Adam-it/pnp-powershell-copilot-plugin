# PnP PowerShell Copilot Plugin

This Copilot Plugin provides a dedicated agent with a set of skills that allows the user to interact with the PnP PowerShell module using natural language. Imagine asking Copilot to "Get all the sites in my tenant" or "Create a new SharePoint site for my project with predefined library and pages" and having it execute the appropriate PnP PowerShell commands on your behalf.

## Capabilities - skills

Below is the list of skills that are currently implemented and are available to the agent:

### Setup

This skill will guide Copilot how to install and configure PnP PowerShell in the user's environment, including setting up an Entra ID app registration for authentication with the necessary permissions for your use case, and signing in to Microsoft 365. It allows you to setup PnP PowerShell either using the interactive login method or using an app registration with certificate or secret.

### Manage Microsoft 365

This skill allows Copilot to execute PnP PowerShell cmdlets to manage Microsoft 365. It can perform a wide range of operations such as creating and managing sites, lists, libraries, pages, permissions, and more. The user can ask Copilot to perform specific tasks like "Create a new communication site for the marketing team" or "Add a new column to the HR list" and it will execute the appropriate PnP PowerShell commands.

### Create Scripts

//.... work in progress 👷🏗️

### Evaluate and update scripts

//.... work in progress 👷🏗️

## Setup - how to use

### VS Code

Use the `Chat: Open Customizations` command and go to plugin tab and click on `Install plugin from source` button. After that provide the source as `adam-it/pnp-powershell-copilot-plugin` and hit enter. You should see additional skills and the new agent available in the Copilot Chat view.
After that, in GitHub Copilot Chat view switch to `PnP PowerShell Agent` to start using the plugin.

### GitHub Copilot CLI

In order to add the plugin to GitHub Copilot CLI simply run the Copilot CLI in terminal and run `/plugin install adam-it/pnp-powershell-copilot-plugin`. After that use the `/agent` command to switch to `PnP PowerShell Agent`.

## Usage examples

Be sure you switched to the `PnP PowerShell Agent` in GitHub Copilot when executing the following examples.

In order to setup PnP PowerShell in your environment, you can ask Copilot:

```
I need to setup my env to use PnP PowerShell. can you help me with that?
```

After that Copilot may ask you to specify the tenant name, how you want to authenticate (interactive login or app registration), and what set of permissions you need.

In order to try out how PnP PowerShell agent may manage your tenant simply ask something like: 

```
Create a new site on my tenant lets say PnP-PowerShell-Tests with a site level app catalog. Change the theme to the site to whatever. 
Create a test-data list on that site with a new custom content type with a custom column and add a few items. Add a new page to the site and add a list web part to this page and make it a home page. 
```

To test out how the agent may create scripts for you, you can ask something like:

```
Create a script that will allow me to create a backup list with all the items of a given list in a given site and then create that given list
```

## Plugin evaluation 

To justify the PnP PowerShell Copilot Plugin positive impact I am performing the following evaluations that are used in order to score and improve the plugin: https://github.com/Adam-it/agent-tests-pnp-powershell.
