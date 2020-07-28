+++
title = "Appendix: Azure Build Agent"
weight = 500
+++

An important part of the CI/CD process for the [wwt-windows-client] repository
is to build the WWT installer [MSI] program. At the moment (July 2020), the
Azure Pipelines [Microsoft-hosted “agents”][ms-agents] do not come equipped
with the tooling needed to build such an installer. So, annoyingly, we need to
create and run our own Windows VM to run these builds. This Appendix documents
how such an agent is set up.

[wwt-windows-client]: https://github.com/WorldWideTelescope/wwt-windows-client/
[MSI]: https://en.wikipedia.org/wiki/Windows_Installer
[ms-agents]: https://docs.microsoft.com/en-us/azure/devops/pipelines/agents/agents#microsoft-hosted-agents

{% warning() %}
These instructions are likely to get out of date quickly. The hope is that
they’ll remain helpful even if they’re not precisely accurate.
{% end %}


# References

- [Microsoft VM scale set agent docs][ms-vmss-docs]
- [Microsoft self-hosted agent docs][ms-selfhosted-docs]
- [Microsoft Visual Studio in VM docs][ms-vs-vm-docs]
- [Stack Overflow #1][stack-overflow-1]

[ms-vmss-docs]: https://docs.microsoft.com/en-us/azure/devops/pipelines/agents/scale-set-agents?view=azure-devops
[ms-selfhosted-docs]: https://docs.microsoft.com/en-us/azure/devops/pipelines/agents/v2-windows
[ms-vs-vm-docs]: https://docs.microsoft.com/en-us/azure/virtual-machines/windows/using-visual-studio-vm
[stack-overflow-1]: https://stackoverflow.com/questions/46570869/vsts-online-building-setup-projects/46761812#46761812


# Initialize Template Machine

It doesn’t seem that there's any good way to set up the VM image
automatically. So, the first step is to create a cloud VM and set it all up by
hand.

1. Install Azure command line tool `az`.
1. `az login` if needed to log in to the WWT Azure subscription.
1. `az account list -o table` to list logged-in subscriptions
1. `az account set -s $SUBSCRIPTION_ID` to set the default subscription
1. `az group create --location westus --name devops-support` to create a
   containing Resource Group.
1. Create the VM:
   ```
   az vm create \
     --resource-group devops-support \
     --name hostedagentbox \
     --image Win2019Datacenter \
     --admin-username wwt \
     --admin-password $PASSWORD \
     --size $VM_SIZE
   ```
   The password must be at least twelve characters, one upper/lower/number/special.
   The machine name cannot be more than 15 characters. For the setup phase, it’s
   nice to use a beefier machine image. (It can be resized later.)
1. Once the machine is created, locate in in the Azure Portal and check parameters.

It should now be possible to RDP into the machine. On Linux, something like:

```
xfreerdp /u:wwt /d:. /v:$IP_ADDR:3389 /size:1280x960
```

Now the graphical setup can begin:

1. Navigate to the [Visual Studio Community][vs-community] downloads page.
1. In IE, go to "Internet Settings", then "Security", and add the current
   webpage as a Trusted website. Otherwise IE doesn’t let you download
   anything. Seriously!
1. Download the `vs_community.exe` installer stub.
1. Run it in an elevated Powershell (which should be the default kind, since
   we’re logged in as the admin user):
   ```
   vs_community.exe --allWorkloads --includeRecommended --passive ^
      --add Microsoft.Net.Component.4.7.SDK ^
      --add Microsoft.Net.Component.4.7.TargetingPack ^ 
      --add Microsoft.Net.Component.4.6.2.SDK ^
      --add Microsoft.Net.Component.4.6.2.TargetingPack ^
      --add Microsoft.Net.ComponentGroup.4.7.DeveloperTools
   ```
   This command cribbed from the [Visual Studio on a VM][ms-vs-vm-docs] docs.
   I am told that it’s important to not start up the graphical interface to
   avoid triggering Visual Studio from wanting a license. If you typed it all
   right, the installer will run and take a while as it downloads several gigs
   of files. If you made a mistake, it will start up and exit pretty quickly
   but without displaying a clear error message.
1. Reboot the machine after the install.
1. Download the [Microsoft Visual Studio Installer Projects][ms-vdproj] Visual
   Studio extension, using the IE Trusted website trick again. As I recall,
   the file is downloaded with a `.zip` extension but can be treated as a
   `.vsix`.
1. Install the extension from the command line, as guided by [this post][vsix-post]:
   ```
   &"C:\Program Files (x86)\Microsoft Visual Studio\2019\Enterprise\Common7\IDE\VSIXInstaller.exe" plugin.zip
   ```
1. Probably doesn't hurt to reboot again.

[vs-community]: https://visualstudio.microsoft.com/vs/community/
[ms-vdproj]: https://marketplace.visualstudio.com/items?itemName=VisualStudioClient.MicrosoftVisualStudio2017InstallerProjects
[vsix-post]: https://developercommunity.visualstudio.com/content/problem/596629/vsixinstaller-from-vs-2019-closing-with-runfromeng.html


# Set up the VM as a Build Agent

Now we need to configure the agent to act as an Azure Pipelines agent. Here we
follow the [Microsoft self-hosted agent docs][ms-selfhosted-docs].

1. Create a Azure DevOps Personal Access Token for a WWT admin user. There is
   a token stored in the WWT LastPass (that is set to expire in July 2021,
   although it's unclear if that will matter).
1. As per the reference docs, set the scope to be “Agent Pools (Read and
   Manage)”.
1. As per the docs, go to the organization settings and get a download link
   for the appropriate agent Windows installer. This link seems to not
   actually be specific to the DevOps organization.
1. Copy the URL onto the VM, use the Trusted site setting on IE to allow
   yourself to download the file, and do so.
1. Finish agent install as per the docs. Set it up to run as a service in the
   "default" agent pool.
1. Go to the Azure DevOps interface for the [WWT organization][azure-devops-wwt].
   The machine should be in the defaut pool!
   
[azure-devops-wwt]: https://dev.azure.com/aasworldwidetelescope/

At this point, upon initial setup, I started testing out the Azure Pipelines
configuration and got it to the point where it all seemed to be working, with
code signing enabled through the [.NET SignClient][sign-client].

[sign-client]: https://github.com/dotnet/SignService#client-configuration


# Convert the VM to work in a scale set

Now we start following the [Microsoft VM scale set agent docs][ms-vmss-docs].

1. “Generalize” the online template VM:
   ```
   &"C:\Windows\System32\sysprep\sysprep.exe" /generalize /oobe /shutdown
   ```
   This will take a long time and shut down the VM.
1. Deallocate the VM:
   ```
   az vm deallocate --resource-group devops-support --name hostedagentbox
   ```
1. Tell Azure to generalize it:
   ```
   az vm generalize --resource-group devops-support --name hostedagentbox
   ```
1. Create a VM image out of it:
   ```
   az image create \
     --resource-group devops-support \
     --name hostedagent-$YYYYMM \
     --source hostedagentbox
   ```
1. Turn it into a VM scale set:
   ```
   az vmss create \
     --resource-group devops-support \
     --name agent-vmss \
     --image hostedagent-$YYYYMM \
     --admin-username wwt \
     --admin-password $PASSWORD \
     --instance-count 2 \
     --disable-overprovision \
     --upgrade-policy-mode manual \
     --load-balancer "" \
     --vm-sku Standard_DS2_v2
   ```
1. Follow the rest of the instructions to wire up the scale set to Azure
   Pipelines. The agent pool is currently called "Custom Windows".
