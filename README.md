# Intune-Setup-for-MacOS
I setup an Intune environment for MacOS from Scratch








Contents
I.	Preparation	2
1.	Apple MDM Push Certificate	2
2.	Groups for Corporate-Owned and BYOD Devices	2
II.	MacOS Configuration Profiles	4
III.	Compliance Policy	5
IV.	Onboarding Defender for Endpoint	7
i.	MDE Onboarding Package	9
ii.	Download the Onboarding Package	10
iii.	Deploy the Onboarding Package with Intune	10
iv.	Publish Microsoft Defender for Endpoint to macOS Devices Using Intune	10
v.	Additional Step (Important): Configure the Default EDR Policy	11
V.	Company Portal and Application Deployment	12







Basic Intune and Defender Setup for MacOS
I.	Preparation
1.	Apple MDM Push Certificate
Configure the Apple MDM Push Certificate (APNs) to establish trust between Intune and Apple devices. This is required before macOS devices can be managed through Intune.
Sources
1.	macOS Management with Intune – The Prologue - https://intuneirl.com/macos-management-with-intune-the-prologue/
2.	How to Get an Apple MDM Push Certificate in Microsoft Intune - https://www.youtube.com/watch?v=ZlJ6-n27hU4
2.	Groups for Corporate-Owned and BYOD Devices
2a. Corporate-Owned macOS Device
For corporate-owned devices, use an assigned device group to automate application and policy deployment.
Example Group: Mac Users - Corporate
Configuration
•	Group Type: Security
•	Membership Type: Assigned
1.	Apple Business Manager Integration
i.	Integrate Apple Business Manager (ABM) with Intune.
ii.	Navigate to Devices → macOS → enrolment → enrolment Program Tokens.
iii.	Select the ABM token.
iv.	Click Profile -> Create Profile.
v.	Configure: 
o	Profile Name: MacOS ABM Enrolment
o	User Affinity: Yes (recommended for user-assigned devices)
o	Authentication Method: Setup Assistant with Modern Authentication
o	Locked Enrolment: Enable
o	Supervised: Yes
2.	Configure Setup Assistant
Select the screens to show or hide during the macOS out-of-box experience for the user.
Commonly Shown Screens
•	Location Services (if required)
•	FileVault Setup

 
Verify that the macOS device is assigned to the enrollment profile by selecting Assign Devices and confirming that the device appears under the MacOS ABM Enrollment profile.
 

Sources
Managing macOS devices with Microsoft Intune - https://www.youtube.com/watch?v=s3mHgPq05wQ
Add a device to Apple Business Manager - https://docs.intunemacadmins.com/complete-guide-macos-deployment/add-a-device-to-apple-business-manager
Integrate Apple Business Manager with Intune - https://docs.intunemacadmins.com/complete-guide-macos-deployment/integrate-apple-business-manager-with-intune


2b. BYOD MacOS Devices
For personally owned devices, use an assigned user group.
Example Group: Mac Users - BYOD
Configuration
•	Group Type: Security
•	Membership Type: Assigned
Add users who are authorized to enrol personal macOS devices.
Purpose: Used to assign applications, configuration profiles, compliance policies, and Conditional Access requirements to BYOD devices.
II.	MacOS Configuration Profiles
MacOS Configuration Profiles in Microsoft Intune are similar to Group Policy Objects (GPOs) in Active Directory, allowing administrators to manage device settings, security controls, and application permissions.
Profiles can be created using:
i.	Settings Catalog (Recommended) – Provides the widest range of macOS management settings.
ii.	Templates – Preconfigured profiles for common scenarios.
iii.	Custom Profiles (.mobileconfig) – Used to deploy vendor-provided configuration profiles.
iv.	Import – Import settings from a JSON configuration file.
Best Practice: Use the Settings Catalog whenever possible, as it provides the most comprehensive and easiest-to-manage macOS configuration options.
Common configuration profiles, security baselines, and templates can be sourced from Microsoft documentation and community GitHub repositories.
Important MacOS Policies to Configure:
•	Platform SSO – Enables Entra ID-based sign-in and identity integration.
•	Microsoft AutoUpdate – Manages updates for Microsoft applications.
•	Software Updates – Controls macOS update settings and deadlines.
•	Restrictions – Applies security and device usage restrictions.
•	FileVault – Enforces disk encryption and recovery key management.
•	Accounts and Login – Configures login, password, and account settings.
•	Defender for Endpoint – Configures Microsoft Defender security and onboarding settings.
Sources
-	https://docs.intunemacadmins.com/baseline-settings-for-intune/settingsoverview
-	https://github.com/SkipToTheEndpoint/OpenIntuneBaseline
-	https://github.com/thenikk/Oceanleaf/tree/main/Intune%20macOS%20Templates
-	Useful - https://intunestuff.com/2024/10/31/macos-intune-policies-guide-to-start/#How_to_Add_a_MacOS_Script_to_Intune
 
Note:
If both corporate-owned and personal (BYOD) devices exist in the environment, some policies may need to be duplicated and reconfigured based on organizational requirements. Corporate policies are typically configured with stricter security controls, while BYOD policies are generally configured with less restrictive settings to balance security and user privacy.
 
III.	Compliance Policy
•	Defines the security requirements a device must meet to be considered compliant.
•	Used to enforce baseline security settings such as device encryption, OS version, password requirements, and device risk level.
Devices that do not meet the defined requirements are marked non-compliant and can be blocked from accessing corporate resources through Conditional Access.  
 
 
IV.	Onboarding Defender for Endpoint
After integrating Intune with Microsoft Defender for Endpoint (MDE), onboard macOS devices to enable advanced threat protection, risk assessment, and enhanced compliance.
Before onboarding, deploy the following pre-requisite MDE configuration profiles:
•	Approve System Extensions
•	Network Filter
•	Full Disk Access
•	Notification Consent
•	Microsoft AutoUpdate (MAU)
Note: This MAU policy updates Defender for Endpoint components and security intelligence. Microsoft 365 app updates are managed through a separate policy.
 

Once these profiles are deployed, configure Defender settings using Intune.
 
There are two methods available:
1.	Configure settings through the Microsoft Defender portal.
2.	Configure settings through the Microsoft Intune admin center.
For this deployment, we will use the Intune method, as all previous configuration profiles and policies were created and managed in Intune.
As described in Step 9b, use the Intune Recommended Profile. Copy the profile contents into a text editor and save the file as - com.microsoft.wdav.xml 1.
Next, create a new macOS Configuration Profile in Intune:
•	Profile type: Templates → Custom
•	Name: MacOS - MDE - WDAV Preferences
•	Configuration profile name: com.microsoft.wdav
•	Configuration profile file: com.microsoft.wdav.xml
Upload the XML file and assign the policy to the appropriate macOS group (Corporate or BYOD).
 
Understanding com.microsoft.wdav
The com.microsoft.wdav.xml file configures Microsoft Defender Antivirus settings on macOS, such as real-time protection, cloud protection, scans, and exclusions. It must be deployed using the configuration name com.microsoft.wdav. Without this profile, Defender can be installed and onboarded, but its antivirus settings are not centrally managed.
i.	MDE Onboarding Package
The MDE Onboarding Package connects a macOS device to the Microsoft Defender for Endpoint service. Installing the Defender app only provides local antivirus protection, while the onboarding package registers the device with the Defender tenant and enables security monitoring.
Without onboarding:
•	Defender runs locally on the device.
•	The device does not appear in the Defender portal.
•	Alerts, telemetry, and device risk information are not available.
With onboarding:
•	The device is registered with Defender.
•	Security telemetry and alerts are sent to the Defender portal.
•	Device risk scores become available for Intune compliance and Conditional Access policies.
ii.	Download the Onboarding Package
1.	In the Microsoft Defender Portal, navigate to Settings → Endpoints → Device Management → Onboarding.
2.	Select: 
o	Operating System: macOS
o	Connectivity Type: Streamlined
o	Deployment Method: Mobile Device Management (MDM)
3.	Click Download Onboarding Package and extract the ZIP file.
 
iii.	Deploy the Onboarding Package with Intune
The Onboarding Package connects the device to Microsoft Defender for Endpoint, enabling telemetry, alerts, and risk information to sync with Defender and Intune.
1.	In Intune Admin Center, go to Devices → macOS → Configuration Profiles and create a new profile:
o	Platform: macOS
o	Profile Type: Templates → Custom
o	Name: MacOS - MDE Onboarding
2.	Upload the onboarding XML file from the extracted package. 
3.	Assign the profile to the appropriate macOS group and click Create.
iv.	Publish Microsoft Defender for Endpoint to macOS Devices Using Intune
After deploying the required configuration profiles and onboarding package, deploy the Microsoft Defender for Endpoint application to managed macOS devices.
1.	In Intune Admin Center, navigate to Apps → macOS → Create.
2.	Select App type: Microsoft Defender for Endpoint (macOS).
3.	Accept the default App Information settings.
4.	Under Assignments, assign the app to the appropriate user or device group.
5.	Review the configuration and click Create.
Once deployed, Microsoft Intune installs Microsoft Defender for Endpoint on the target macOS devices. After onboarding is complete, the devices appear in the Defender portal, begin reporting security telemetry, and provide device risk signals that can be leveraged by Intune compliance policies and Conditional Access controls.
v.	Additional Step (Important): Configure the Default EDR Policy
If no EDR policy exists, create one:
•	Platform: macOS
•	Profile: Endpoint Detection and Response
•	Policy Name: MacOS Default EDR Policy
•	Assign the policy to the target group and complete the policy creation wizard.
 
While WDAV provides antivirus and threat prevention, EDR delivers advanced threat detection, alerting, investigation, and automated response capabilities. Together, they provide comprehensive endpoint protection on macOS.
Components Overview:
•	Defender App = Protection engine
•	com.microsoft.wdav.xml = Antivirus configuration
•	Onboarding Package = Connects the device to Defender service
•	EDR Policy = Advanced detection and response capabilities
Sources
1.	Deploy Microsoft Defender for Endpoint on macOS with Microsoft Intune - https://learn.microsoft.com/en-us/defender-endpoint/mac-install-with-intune
2.	Enroll MacOS in Microsoft Defender - https://docs.intunemacadmins.com/complete-guide-macos-deployment/enroll-macos-in-microsoft-defender
3.	How to Deploy Defender for Endpoint on macOS Using Intune - https://www.youtube.com/watch?v=yn5nI8lkhps&list=PLKROqDcmQsFmie_lKJe-kHOfh4uWyqA9h&index=4&t=606s
4.	Mobileconfig files (Official Microsoft Repo) - https://github.com/microsoft/mdatp-xplat/tree/master/macos/mobileconfig/profiles
V.	Company Portal and Application Deployment
Deploying Microsoft Edge, Microsoft 365 Apps, and Microsoft Defender for Endpoint is straightforward, as they are natively available in Intune.
For other applications, deployment options include .pkg, .dmg, Line-of-Business apps, and shell scripts.
.pkg installers are generally preferred because they are the simplest and most reliable deployment method.
 
Example: Deploy Google Chrome as a Line of Business (.pkg) App
1.	Download the Google Chrome macOS (.pkg) installer from Google's website.
 
2.	In Intune Admin Center, go to Apps > macOS > Add and select Line-of-business app.
 
3.	Upload the downloaded .pkg file.
4.	Under App information, enter the Name, Description, and Publisher. Configure the Minimum Operating System requirement and disable Install as managed if no additional management/customization is required.
 
5.	Leave Scope tags as default. Under Assignments, add the target group as Required for automatic installation and Available for enrolled devices to make the app available in the Company Portal.
Example: Deploy Firefox as a macOS App (.dmg)
1.	In Apps > macOS, click Create and select macOS app (.dmg). 
2.	Upload the downloaded Firefox .dmg file. 
3.	Under App information, enter the Name, Description, and Publisher, and upload a PNG logo to display in the Company Portal. 
4.	Under Requirements, select the minimum operating system. 
5.	Under Detection rules, obtain the App Bundle ID and App Version by opening the DMG package with 7-Zip, navigating to Firefox.app > Contents, and opening Info.plist. Search for CFBundleIdentifier and CFBundleShortVersionString, then enter the corresponding values in Intune.
 
   
 
6.	Leave Scope tags as default. 
7.	Under Assignments, add the target group as Required for automatic installation and Available for enrolled devices to make the app available in the Company Portal. 
8.	Review the configuration and click Create.


Alternate Option:
1.	For application deployment, if a standard PKG or DMG installer is not available, look for vendor-provided deployment resources or trusted community repositories such as GitHub. Where required, use deployment scripts to automate application installation and uninstallation.
 
2.	Use IntuneBrew (One-click) - https://www.intunebrew.com/apps
Sources
1.	How to Deploy Mac Apps with Intune - https://www.youtube.com/watch?v=_T7SY59D7b4
2.	Microsoft GitHub Repo - https://github.com/microsoft/shell-intune-samples/tree/master/macOS/Apps
