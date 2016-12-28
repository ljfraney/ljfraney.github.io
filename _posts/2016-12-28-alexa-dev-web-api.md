---
layout: post
title: Alexa Development with ASP.Net Web API
description: "Developing Alexa Skills with ASP.Net Web API"
modified: 2016-12-28
tags: [alexa, .net, c#, asp.net, WebAPI]
image:
  feature: AlexaWebAPI/EchoDotNet.jpg
---

Like many new Alexa developers, I created my first Alexa Skill, [C# Quiz](http://bit.ly/CSharpQuiz "C# Quiz Alexa Skill on Amazon"), by downloading and modifying the [Trivia Game Template](https://developer.amazon.com/blogs/post/TxDJWS16KUPVKO/new-alexa-skills-kit-template-build-a-trivia-skill-in-under-an-hour "New Alexa Skills Kit Template: Build a Trivia Skill in under an Hour"). It is backed by an AWS Lambda function, and I chose to write it in node.js, because Amazon has so graciously created the [node.js SDK for Alexa Skills Kit](https://github.com/amzn/alexa-skills-kit-js "GitHub Page for node.js ASK SDK"). This was a great way to get my feet wet, and I would recommend it to anyone writing their first Alexa Skill. It will allow you to create and host your skill quickly, and it will help you learn your way around the [Developer Portal](https://developer.amazon.com/alexa "Amazon Developer Portal"). You will get to know the interaction model, and you'll learn the structure of the requests and responses that you will have to handle in your code. Most importantly, it will familiarize you with the certification process, and enable you to get your first skill in the store. Once you have a skill in the store, you will be totally excited about working on the next one. You know, the one that will make the world a better place!

After completing my first skill, I decided to shift my focus to the familiar. Learning a new technology can be a difficult task. Learning two new technologies at once is downright overwhelming. I've spent much of the last decade developing in C# and ASP.Net, and I needed to spend some of that Azure credit that came with my MSDN subscription! Of course, you could host node.js in Azure. You could also host .Net in AWS, but not yet in a Lambda Function.

## Setting Up your Development Environment to Accept Requests from Alexa with a Self-Signed Certificate

For the best development experience possible, you are going to want to test your API by running it locally, allowing requests from your skill to be handled by the code running on your development machine. This way you can set breakpoints in your code to see what the heck is happening. There are several steps you can take to put this puzzle together, some required, some recommended.

1. You are going to need to forward port 443 traffic to your development machine. **This is a show-stopper if you are on a network where you don't have the ability to perform this step.** There are plenty of articles on how to set this up, so I will direct you to [Google](https://google.com "Google Search") to research how to do it for your specific router. Since this article is specifically about using IIS Express, I should mention another caveat here. Once you've enabled SSL for your app by following the instructions in "[Enabling your Web API App to Use SSL](#enableSSL)" below, it will be assigned a port number, and it won't be 443. It will be something like port 44395. When you are setting up port forwarding, make sure that you forward external port 443 to the specific internal port that your app uses in IIS Express (i.e. 44395).

2. This step is suggested, but not required. I recommend that you have a domain name that points to your development machine. You can use a free service like [noip.com](http://www.noip.com "Free Dynamic DNS from noip.com") to create a domain that you can update with your machine's IP address at any time. Most of these services offer a client app that runs on your machine, updating the service with your new IP when it changes. If you already have a domain name, or you want to register your IP address with Amazon, you can skip this step. If you are already using a dynamic DNS service to access your home network from the Internet, you might consider adding another hostname for Alexa development. This way, you can potentially develop when you are away from home without changing the IP address of your existing hostname, provided you can accomplish step #1.

3. You are going to set up your Web API project to use SSL. This is pretty straight forward, and I'll describe it in detail [later in the post](#enableSSL "Enabling your Web API App to Use SSL").

4. You are going to need a certificate. You can buy one. You can get a free one. I hear [letsencrypt.org](https://letsencrypt.org "Let's Encrypt") is nice, but I haven't tried it yet. Or, you can create your own certificate. That's the method I'm going to describe in this post, but if you intend to use the Audio SSML tag, you will have to use a certificate from a trusted [Certificate Authority](https://en.wikipedia.org/wiki/Certificate_authority "Wikipedia article on certifite authority"). If you are wondering if your CA is trusted by Amazon, you can refer to the [Mozilla Included CA Certificate List](https://wiki.mozilla.org/CA:IncludedCAs, "Mozilla Included CA Certificate List"). That is the list that Amazon uses.

### Creating a Self-Signed Certificate to Test your Skill

Amazon has [instructions for creating a self-signed certificate](https://developer.amazon.com/public/solutions/alexa/alexa-skills-kit/docs/testing-an-alexa-skill#h2_sslcert "Amazon Developer Portal - Testing a Custom Skill") using [OpenSSL](https://www.openssl.org/source "OpenSSL Home Page") on the Linux platform. There is also a good [post from freebusy.io](https://freebusy.io/blog/getting-started-with-alexa-app-development-for-amazon-echo-using-dot-net "Getting started with Alexa App development for Amazon Echo using .NET on Windows") about this that you will likely find mentioned in other articles and blogs. However, it uses the MakeCert.exe utility that comes with older versions of the Windows <abbr title="software development kit">SDK</abbr>. [MakeCert is deprecated](https://msdn.microsoft.com/en-us/library/windows/desktop/aa386968(v=vs.85).aspx "MakeCert on MSDN"), and doesn't support the [Subject Alternative Name Extension](https://en.wikipedia.org/wiki/Subject_Alternative_Name "Wikipedia Article about Subject Alternative Name"). FreeBusy's article was written in June, 2015, before Amazon required the use of Subject Alternative Name. Below, I'll describe two methods you can use to create a certificate with Windows that meets Amazon's latest requirements. If you haven't already enabled Bash on Ubuntu on Windows 10, I would recommend the Powershell approach. Otherwise, both methods are equally painful, so pick whichever one interests you most.

#### Method 1: Using Powershell's New-SelfSignedCertificate Cmdlet to Create a Certificate

Since MakeCert is deprecated, Microsoft recommends using the [New-SelfSignedCertificate Powershell Cmdlet](https://technet.microsoft.com/library/hh848633 "New-SelfSignedCertificate Powershell Cmdlet on MSDN") to create certificates. Use the following command in PowerShell 3.0 or higher to create your certificate and install it in the Local Computer certificate store in one easy step. You must run PowerShell as Administrator to do this.

`New-SelfSignedCertificate -DnsName myawesomedevdomainname.com -CertStoreLocation cert:\LocalMachine\My`

![PowerShell Image 1]({{ site.url }}/images/AlexaWebAPI/PowerShellCreateCert.png)

I'll describe how to retrieve it later in the post, but this would be a good time to copy the thumbprint. You'll need it when you get to "[Allowing Incoming Requests on Port 443 in HTTP.SYS and Windows Firewall](#httpsysandwindowsfirewall)". Let's go find that certificate. Open the Microsoft Management Console (MMC). You can usually just press the Start button (aka Window Key), type *mmc.exe*, and press *Enter*. Once you are in <abbr title="Microsoft Management Console">MMC</abbr>, go to the *File* menu and select *Add/Remove Snap-in*. Select *Certificates* and click the *Add* button.

![MMC Image 1]({{ site.url }}/images/AlexaWebAPI/mmc1.png)

Select *Computer account* and click *Next*.

![MMC Image 2]({{ site.url }}/images/AlexaWebAPI/mmc2.png)

Select *Local computer* and click *Finish*. Click *OK* to dismiss the *Add or Remove Snap-ins* dialog.

![MMC Image 3]({{ site.url }}/images/AlexaWebAPI/mmc3.png)

Expand *Certificates (Local Computer) > Personal* and click on *Certificates*.

![MMC Image 4]({{ site.url }}/images/AlexaWebAPI/mmc4.png)

Now you need to get the certificate to upload into the Developer Portal. Right-click on your certificate, choose *All Tasks > Export...*.

![MMC Image 5]({{ site.url }}/images/AlexaWebAPI/mmc5.png)

Click *Next*, select *No, do not export the private key*, and click *Next* again.

![MMC Image 6]({{ site.url }}/images/AlexaWebAPI/mmc6.png)

Select *Base-64 encoded X.509 (.CER)* and click *Next*.

![MMC Image 7]({{ site.url }}/images/AlexaWebAPI/mmc7.png)

Give it a file name. I would suggest specifying a location, because the default location is C:\Windows\System32. Click *Next*, and then *Finish*.

![MMC Image 8]({{ site.url }}/images/AlexaWebAPI/mmc8.png)

Next, open the exported file in a text editor (notepad will work), and copy the contents. Be sure to include the BEGIN CERTIFICATE and END CERTIFICATE lines.

![Certificate in Notepad]({{ site.url }}/images/AlexaWebAPI/notepad1.png)

This is what you will paste into the Developer Portal so that your Alexa Skill can talk to your machine. More on that below.

#### Method 2: Using Bash on Ubuntu on Windows to Create a Certificate with OpenSSL

If you are running 64-bit Windows 10 with the Anniversary Update, you can create a certificate using Bash on Ubuntu on Windows. Here is an [article from How-to-Geek](http://www.howtogeek.com/249966/how-to-install-and-use-the-linux-bash-shell-on-windows-10 "How to Install and Use the Linux Bash Shell on Windows 10 from How-to-Geek") describing how to enable this Windows feature. Using this method, you can almost follow [Amazon's instructions](https://developer.amazon.com/public/solutions/alexa/alexa-skills-kit/docs/testing-an-alexa-skill#h2_sslcert "Amazon Developer Portal - Testing a Custom Skill") for creating the certificate. I say "almost" because Amazon's instructions will result in the creation of two files, one that is your private key, and one that is your certificate. You will then need to combine the two into one file, which is as easy as copying the contents of the private key file into the certificate file. Here are instructions for creating one file that contains both your private key, and your certificate without the extra step.

Open Bash, and create a directory for your certificate, unless you want to store it in your home directory. Change to the new directory and open a text editor to create your configuration file. I'm naming mine *configuration.cnf*.

![Bash Image 1]({{ site.url }}/images/AlexaWebAPI/BashCreateCertConfiguration.png)

Enter the configuration info, replacing your country, state, city, organization, skill name, and domain name. Then save the file.

![Bash Image 2]({{ site.url }}/images/AlexaWebAPI/BashCreateCertConfiguration2.png)

Use the following OpenSSL command to create a single file containing both the certificate and the private key.

`openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout cert.pem -out cert.pem -config configuration.cnf`

![Bash Image 3]({{ site.url }}/images/AlexaWebAPI/BashCreateCert.png)

Use the next command to create a .pfx file that can be imported to your Local Computer certificate store in Windows. Name it whatever you like. Also, use whatever password you want, but remember it for the next step.

`openssl pkcs12 -export -out cert.pfx -in cert.pem -name "Alexa Dev Cert" -passout pass:mypassword`

![Bash Image 4]({{ site.url }}/images/AlexaWebAPI/BashCreateCert2.png)

As you can see, you now have a file named *cert.pfx* that is ready to import into your Local Computer certificate store. But where the heck is it? Well, there are two ways to get to it. First, you could copy it from Linux to Windows. Your Windows drives are mounted in Linux under the following directory. (You will need to go back two directories from your home directory before you will see *mnt*.)

`/mnt/driveletter$ (i.e. /mnt/c$)`

Or, you could browse to the folder in File Explorer, by navigating to the following path.

`C:\Users\{yourusername}\AppData\Local\lxss\home\{yourusername}`

Some of those folders are hidden, so you may have to type them in manually in File Explorer. The lxss folder isn't even visible when I have "Hidden Items" selected. Whichever way you choose to get access to your new .pfx file, right-click on it in File Explorer and select *Install PFX*.

![Install PFX 1]({{ site.url }}/images/AlexaWebAPI/InstallPFX.png)

Select *Local Machine* and click *Next*. Then click *Next* again to confirm the location of the file you are importing.

![Install PFX 2]({{ site.url }}/images/AlexaWebAPI/InstallPFX2.png)

Enter the password that you used in the *openssl pkcs12* step above. You can decide whether to mark the certificate as exportable. I usually do, but you don't have to.

![Install PFX 3]({{ site.url }}/images/AlexaWebAPI/InstallPFX3.png)

You should be able to select *Automatically select the certificate store based on the type of certificate*. You want the certificate to be in the *Local Computer > Personal > Certificates* store. That is where it should put it when you select this option.

![Install PFX 4]({{ site.url }}/images/AlexaWebAPI/InstallPFX4.png)

Now, open the *cert.pem* file with a text editor. It should be in the same folder as the .pfx file you just imported. Copy the certificate. Be sure to include the BEGIN CERTIFICATE and END CERTIFICATE lines. DO NOT include the private key. That is for your eyes only!

### Entering your Certificate in the Developer Portal

Sign in to the [Developer Portal](https://developer.amazon.com "Amazon Developer Portal") and open the settings for your Alexa Skill. If you haven't created your skill, refer to Amazon's article [Requirements to Build a Skill](https://developer.amazon.com/public/solutions/alexa/alexa-skills-kit/docs/requirements-to-build-a-skilld "Requirements to Build a Skill"). Make sure that you've selected *HTTPS* in the *Endpoint* section under *Configuration*. Select your geographic region, and enter the URL of your endpoint. This should be the fully qualified route to your Web API endpoint that will handle Alexa Requests. So, if your path is */api/Alexa* or something similar, be sure to include that here.

![Devloper Portal 1]({{ site.url }}/images/AlexaWebAPI/DevPortal1.png)

Go to the *SSL Certificate* section and select *I will upload a self-signed certificate in X.509 format*. Paste the contents of the certificate you created and click *Save*. You should see a message that reads *Successfully updated the SSL certificate*. It is possible that the checkmark won't turn green. And, if you browse away and return to this page, you will not see the certificate anymore. From my experience, if you saw the *Successfully updated the SSL certificate* message, and your endpoint is set up correctly, you are all set.

![Devloper Portal 2]({{ site.url }}/images/AlexaWebAPI/DevPortal2.png)

### Enabling your Web API App to Use SSL<a name="enableSSL"></a>

Your Alexa Skill can only make requests over <abbr title="secure sockets layer">SSL</abbr> on port 443. By default, your application isn't configured to use SSL. To set this up, open your Web API Project in Visual Studio. Select the Project in Solution Explorer, and go to the Properties window. This part can be confusing. You don't want to open the Properties node under your project. You also don't want to right-click on the project and select Properties from the context menu. Instead, you want to click once on the Project, and then open the Properties window in Visual Studio. Usually, it's pinned to the right side of Visual Studio. If not, you can click on the *View* menu, and select *Properties Window*, or you can just press F4. From there, set *SSL Enabled* to *True*. Below this setting, you should now see a property called *SSL URL* that includes a five-digit port that starts with 443. In the image below, the port is 44395. This is the port that you should use as the internal port when you set up port forwarding in your router.

![Visual Studio Project Properties 1]({{ site.url }}/images/AlexaWebAPI/VSProperties1.png)

### Allowing Incoming Requests on Port 443 in HTTP.SYS and Windows Firewall<a name="httpsysandwindowsfirewall"></a>

HTTP.SYS handles all incoming HTTP traffic in Windows. You need to enable requests on the port that your app is using. You can do this at the command line with the *netsh* command below. You must run as Administrator for this to work. Notice that I've enabled port 44395 below. Replace this with the port that was assigned to your application in Visual Studio.

`netsh http add urlacl url=https://localhost:44395/ user=Everyone`

You also need to tell HTTP.SYS about the certificate that we created. Replace *\<thumbprint\>* with your certificate's thumbprint. If you followed *Method 1* and created your certificate with Powershell, you may have already copied your thumbprint. If you followed *Method 2*, or you didn't copy your thumbprint already, go back to the Local Computer certificate store in mmc.exe and find your certificate. Double-click on the certificate, go to the Details tab, and scroll down to the Thumbprint field. Copy the value, and then remove all of the spaces. The appid can be any GUID.

`netsh http add sslcert ipport=0.0.0.0:44395 certhash=<thumbprint> appid={12345678-1234-1234-1234-AABBCCDDEEFF}`

If you are using PowerShell, you will need to escape the curly braces. One way to do this is to use the -% option.

`netsh http add sslcert ipport=0.0.0.0:44395 certhash=<thumbprint> --% appid={00112233-4455-6677-8899-AABBCCDDEEFF}`

Windows Firewall also needs to be configured to allow incoming traffic over this port. You can use the Windows Firewall <abbr title="graphical user interface">GUI</abbr> to set this up, or you can use the following command, again replacing port 44395 with the port that your app is using.

`netsh advfirewall firewall add rule name="HTTPS" dir=in action=allow protocol=TCP localport=44395`

### Setting up IIS Express to Handle External Requests

Visual Studio 2015 ships with IIS Express 10.0. IIS Express settings are stored in a file named *applicationhost.config*. Unlike previous versions of IIS, version 10's *applicationhost.config* is found in the *\\.vs\\config\\* folder under your application's folder. Open *applicationhost.config* in a text editor, find your application under the *sites* node, and add your hostname like the example below. If you skip this step, you will get the error "There was an error calling the remote endpoint, which returned HTTP 400 : Bad Request" when testing your skill from the Developer Portal. Testing from a tool like [Postman](https://www.getpostman.com "Postman Home Page") will give you the more useful error "HTTP Error 400. The request hostname is invalid." If you are like me, and you tend to blow away the .vs folder as a first measure whenever something goes wrong with Visual Studio, you will want to remember to fix this.

```xml
<system.applicationHost>
    <sites>
        <site name="MyAlexaSkillAPI" id="1" serverAutoStart="true">
            <application path="/">
                <virtualDirectory path="/" physicalPath="%IIS_SITES_HOME%\MyAlexaSkillAPI" />
            </application>
            <bindings>
                <binding protocol="http" bindingInformation=":8080:localhost" />
                <binding protocol="https" bindingInformation="*:44395:localhost" />
                <!-- Add your custom hostname here -->
                <binding protocol="https" bindingInformation=":44395:myawesomedevdomainname.com" />
            </bindings>
        </site>
    </sites>
</system.applicationHost>
```

### Write your API

There are several ways you could start writing your API. You could roll your own, if you enjoy pain, or you could use one or more of these resources.
 
1. [Bob Lautenbach](http://braneworks.com "Braneworks") created a project called [AlexaASKNetTemplate](https://github.com/boblautenbach/AlexaASKNetTemplate "AlexaASKNetTemplate GitHub Page"). It includes a Visual Studio template that enables you to create a new Alexa Skill API from the New Project dialog in Visual Studio. It has the request and response objects mapped to classes, and boilerplate code for handling most common requests. It also has the code for verifying the certificate that is used to make requests to your app, which will be required to pass Amazon's certification process for your skill.![Echo Template]({{ site.url }}/images/AlexaWebAPI/EchoTemplate.png)

2. [Walter Quesada](http://qnovations.com "Qnovations") authored a Pluralsight course called [Developing Alexa Skills for Amazon Echo](https://www.pluralsight.com/courses/amazon-echo-developing-alexa-skills). Pluralsight gives you a 10-day trial when you sign up. You can cancel online at any time without having to pick up the phone. If, like me, you get busy and don't finish the course before your 10 days is up, you are converted to a $29/month subscription. If you cancel during that month, your account will remain active until the end of the month that you've paid for.

3. [FreeBusy](https://freebusy.io "FreeBusy's Website") created an [SDK similar to the Java ASK SDK](https://github.com/AreYouFreeBusy/AlexaSkillsKit.NET "AlexaSkillsKit.NET GitHub Page") created by Amazon. I haven't tried this yet, but it looks it looks promising.

Now, Go Code!