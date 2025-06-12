# Windows Installer

Windows Installer, aka MSI, is an installer format created by Microsoft.  They store the files to be installed as well as the required installation steps, so they may also act as containers in the initial access taxonomy.  MSI's can be created using [Visual Studio](https://visualstudio.microsoft.com/) and the [Installer Projects](https://marketplace.visualstudio.com/items?itemName=VisualStudioClient.MicrosoftVisualStudio2022InstallerProjects) extension.  Begin by creating a new _Setup Project_.

<figure><img src="https://lwfiles.mycourse.app/66e95234fe489daea7060790-public/9e8ac9af38fcbbb279058fc6842f232f.png" alt="" width="100%"><figcaption></figcaption></figure>

This will create a new blank installer project.  The default view are of the files that will be packaged into the installer.  The next step is to add your payload dependencies.  The location for these can be in application directory (i.e. Program Files), the user's desktop, or the start menu.   For this example, let's put the payload executable into the application directory.  Right-click on whichever location you want, select **Add > File** and add a payload, such as `C:\Payloads\http_x64.exe`.

<figure><img src="https://lwfiles.mycourse.app/66e95234fe489daea7060790-public/a7b1ea3f4f561dcecd49753b154b5aec.png" alt="" width="100%"><figcaption></figcaption></figure>

You can change several properties of the file such as whether you want it to be hidden after it's dropped onto the machine, and its new filename.

<figure><img src="https://lwfiles.mycourse.app/66e95234fe489daea7060790-public/689d0d05934a3bc6a158f729c370be19.png" alt="" width="100%"><figcaption></figcaption></figure>

Next, we need to add an action to ensure the payload gets executed during the installation process.  Right-click the project in the Solution Explorer and select **View > Custom Actions**.

<figure><img src="https://lwfiles.mycourse.app/66e95234fe489daea7060790-public/852953ced5f94e6d8487b67c1de822a0.png" alt="" width="100%"><figcaption></figcaption></figure>

Right-click the **Install** step and select **Add Custom Action**.  Then from the pop-up, select the **application** folder and then select the payload file.

<figure><img src="https://lwfiles.mycourse.app/66e95234fe489daea7060790-public/2e8c1b23808594c99828ba0086009a39.png" alt="" width="100%"><figcaption></figcaption></figure>

Since this payload is 64-bit, set the **Run64Bit** to **True** in the action properties.  You can also modify the command line arguments that get passed to the payload to make it look more legitimate.

<figure><img src="https://lwfiles.mycourse.app/66e95234fe489daea7060790-public/fc550e4cc804f764e2168d690a25dd9d.png" alt="" width="100%"><figcaption></figcaption></figure>

You can also modify the project properties to dress up the installer with a company name, product name, and the target platform, etc.  This will change things like the name of the directory that gets created in Program Files, and the name of the installation in places like the Control Panel.

<figure><img src="https://lwfiles.mycourse.app/66e95234fe489daea7060790-public/4b91038ece4396da3b83ba5fd8816402.png" alt="" width="100%"><figcaption></figcaption></figure>

After building the project (**Build > Build Solution**) you will get two files - a `.exe` and a `.msi`.  The EXE is just a wrapper that runs the MSI, so it doesn't need to be included when delivering to a victim.

<figure><img src="https://lwfiles.mycourse.app/66e95234fe489daea7060790-public/26ac3af920ab133bfe699f55e91e1045.png" alt="" width="100%"><figcaption></figcaption></figure>

