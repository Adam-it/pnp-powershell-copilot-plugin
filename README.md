# PnP PowerShell Copilot Plugin

This Copilot Plugin provides a dedicated agent with a set of skills that allows the user to interact with the PnP PowerShell module using natural language. Imagine asking Copilot to "Get all the sites in my tenant" or "Create a new SharePoint site for my project with predefined library and pages" and having it execute the appropriate PnP PowerShell commands on your behalf.

> Tip: For the best experience please use the plugin with 'Claude Sonnet 4.6' or 'Claude Opus 4.6' or similar or newer and better models.

## Capabilities - skills

Below is the list of skills that are currently implemented and are available to the agent:

### Setup

This skill will guide Copilot how to install and configure PnP PowerShell in the user's environment, including setting up an Entra ID app registration for authentication  with the necessary permissions for your use case, and signing in to Microsoft 365. It allows you to setup PnP PowerShell either using the interactive login method or using an app registration with certificate or secret.

### Manage Microsoft 365

This skill allows Copilot to execute PnP PowerShell cmdlets to manage Microsoft 365. It can perform a wide range of operations such as creating and managing sites, lists, libraries, pages, permissions, and more. The user can ask Copilot to perform specific tasks like "Create a new communication site for the marketing team" or "Add a new column to the HR list" and it will execute the appropriate PnP PowerShell commands.

### Create Scripts

This skill enables Copilot to generate production-ready PowerShell scripts that use PnP PowerShell. The user can describe the desired functionality of the script in natural language, and Copilot will scaffold a PowerShell script with parameters, error handling, and logging based on the user's requirements.

## Setup - how to use

### VS Code

> Note: The below setup for VS Code might be change quite soon

Currently VS Code does not support installing plugin from other GitHub Repo then the approved or added plugin marketplaces, this might change very soon, but for now in order to use the plugin you need to clone this repository locally. Then open VS Code and use `Chat: Plugin Locations` setting to add the path to the cloned repository. After that you should be able to see `PnP PowerShell Agent` in the list of available plugins when using `Chat: Open Customizations` view, and together with this plugin you should see the related skills and agent.
In GitHub Copilot Chat view switch to `PnP PowerShell Agent` to start using the plugin.

### GitHub Copilot CLI

// TODO

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
Create a test-data list on that site with a new custom content type with a custom column and adda few items. Add a new page to the site and add a list webpart to this page and make it a home page. 
```

To test out how the agent may create scripts for you, you can ask something like:

```
Create a script that will allow me to create a backup list with all the items of a given list in a given site and then create that given list
```