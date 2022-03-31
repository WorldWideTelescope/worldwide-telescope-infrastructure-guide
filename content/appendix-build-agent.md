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

1. First-time setup steps, if needed:
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
     --size Standard_D2_v3 \
     --admin-username wwt \
     --admin-password $PASSWORD
   ```
   The password must be at least twelve characters, one upper/lower/number/special.
   The machine name cannot be more than 15 characters. For the setup phase, it’s
   nice to use a beefier machine image.
1. Note the public IP address that is printed out by the creation step.

It should now be possible to RDP into the machine. On Linux, something like:

```
xfreerdp /u:wwt /v:$IP_ADDR:3389 /size:1280x960
```

or, if you're on a HiDPI display, perhaps with:

```
... /size:2560x1600 /scale:180
```

In one instance it looked like the program hung, but I just needed to hit Enter
(?). Now the graphical setup can begin:

1. If prompted upon login, turn off network discoverability.
1. In the Server Manager, go to "Local Server", then click on "IE Enhanced
   Security Configuration". Turn the settings to off. Otherwise, IE literally
   won't allow you to download files without jumping through a lot of hoops.
   ([Ref][ie-esc]).
1. It seems to now be necessary to install Microsoft Edge in order to download
   VS2019. Go to the [Microsoft Edge download page][ms-edge] and install it.
   (Note: copy-paste of links into the RDP screen should work!) Select "Don't
   use" when IE prompts about adopting recommended security policies.
1. Now use Edge to navigate to the [Visual Studio 2019 Community][vs-community]
   downloads page. Login to a Microsoft account will be necessary; password
   copy-paste out of the host machine FTW.
1. Download the `vs_community_*.exe` installer stub.
1. Rename that file to just `vs_community.exe` (just to make copy-paste easier
   in the next step).
1. Run it in an elevated Powershell (which should be the default kind, since
   we’re logged in as the admin user):
   ```
   .\vs_community.exe --allWorkloads --includeRecommended --passive `
      --add Microsoft.Net.Component.4.7.SDK `
      --add Microsoft.Net.Component.4.7.TargetingPack `
      --add Microsoft.Net.Component.4.6.2.SDK `
      --add Microsoft.Net.Component.4.6.2.TargetingPack `
      --add Microsoft.Net.ComponentGroup.4.7.DeveloperTools
   ```
   This command cribbed from the [Visual Studio on a VM][ms-vs-vm-docs] docs.
   I am told that it’s important to not start up the graphical interface to
   avoid triggering Visual Studio from wanting a license. If you typed it all
   right, the installer will run and take a while as it downloads several gigs
   of files. If you made a mistake, it will start up and exit pretty quickly
   without displaying a clear error message.
1. Reboot the machine after the install. (Not 100% sure but I strongly suspect
   this is necessary.)
1. Download the [Microsoft Visual Studio Installer Projects][ms-vdproj] Visual
   Studio extension. If you download with IE instead of Edge, it lands with a
   `.zip` extension rather than `.vsix`.
1. Install the extension from the command line:
   ```
   &"C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\Common7\IDE\VSIXInstaller.exe" InstallerProjects.vsix
   ```

[ms-edge]: https://www.microsoft.com/en-us/edge/business/download
[vs-community]: https://visualstudio.microsoft.com/vs/older-downloads/
[ms-vdproj]: https://marketplace.visualstudio.com/items?itemName=VisualStudioClient.MicrosoftVisualStudio2017InstallerProjects
[ie-esc]: https://medium.com/tensult/disable-internet-explorer-enhanced-security-configuration-in-windows-server-2019-a9cf5528be65

# Convert the VM to work in a scale set

Now we start following the [Microsoft VM scale set agent docs][ms-vmss-docs].

1. “Generalize” the online template VM:
   ```
   &"C:\Windows\System32\sysprep\sysprep.exe" /generalize /oobe /shutdown
   ```
   This will shut down the VM. The docs suggest that the process might take
   a very long time, but in at least some cases it’s pretty quick.
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
     --source hostedagentbox \
     --name hostedagent-$YYYYMM
   ```
1. Turn it into a VM scale set. Note that this command seems to work even in
   steady-state, when the scale set already exists:
   ```
   az vmss create \
     --resource-group devops-support \
     --name agent-vmss \
     --instance-count 2 \
     --disable-overprovision \
     --upgrade-policy-mode manual \
     --load-balancer "" \
     --vm-sku Standard_D2s_v3 \
     --admin-username wwt \
     --admin-password $PASSWORD \
     --image hostedagent-$YYYYMM
   ```
   Note: I'd like to use the `Standard_D2_v3` VM SKU, which doesn't call for
   fancy "premium disk", but when I tried that I got an error about it being
   required anyway.
1. Finally, delete the VM:
   ```
   az vm delete -g devops-support -n hostedagentbox
   ```


# Older Notes

If you want to do a test build in the VM, you’ll need to apply [the fix for the
HRESULT = 8000000A error][hresult-error]:

```
cd "C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\Common7\IDE\CommonExtensions\Microsoft\VSI\DisableOutOfProcBuild"
.\DisableOutOfProcBuild.exe
```

[hresult-error]: https://stackoverflow.com/a/41788791/3760486

We need to apply the same fix on-the-fly in the VMs since the fix is
user-specific and the Azure provisioning uses a different user.

If you want the VM to to act as an Azure Pipelines build agent on its own,
follow the [Microsoft self-hosted agent docs][ms-selfhosted-docs]. However,
we’re setting up the image to run in a VM scale set, in which case the agent
will automatically be provisioned upon VM boot. So for the main workflow you
should **not** install the agent.

If setting up from scratch, after creating the scale set you need to follow the
rest of [the instructions][ms-vmss-docs] to wire it up to Azure Pipelines. The
agent pool is currently called "Custom Windows".
