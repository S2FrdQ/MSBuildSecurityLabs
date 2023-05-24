# Introduction

APIs are now one of the top attack vectors, and it is critical to ensure you have full API security coverage from the start of development all the way to production. In this lab, we will provide a hands-on experience using Defender for Clouds new API security feature sets, Defender for APIs, to provide security for APIs published in Azure API management. The goal of this lab to help gain practical skills and insights into how to protect your APIs that are published in Azure API Management platform.

During the lab experience you will publish an API using Azure API Management Service, remediate security findings through infrastrucure as code (IaC) scanning in CI/CD, and identify high risk security misconfigurations for published APIs. If you are interested in additional developer focused security feature sets in Microsoft Defender for Cloud, please attend the deep dive session of shift left security with Defender for Devops.

By the end of this session, you'll have a clear understanding of how to use Defender for APIs to provide full lifecycle API protection in Azure, helping you safeguard your cloud applications against attacks.

## Prerequisite ##
- Create a resource group (In this lab, we will use RG1) on Azure portal: https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/manage-resource-groups-portal


## MODULE 1: PUBLISH AN API WITHIN API MANAGEMENT ##

In this module, we will be publishing APIs within Azure API Management. Azure API Mangement provides a method to publish any internal and external APIs in a secure and consistent method. With Azure API Management enabled, all requests from client applications first reach the API gateway (a component of API management), which then forwards them to respective backend services. The API gateway acts as a facade to the backend services, allowing API providers to abstract API implementations and evolve backend architecture without impacting API consumers. The gateway enables consistent configuration of routing, security, throttling, caching, and observability. During this lab, we will not be configuring typical security policies in Azure API Management (such as rate-limiting, jwt-token validation); however, if you are interested in learning more, please visit here: https://learn.microsoft.com/en-us/azure/api-management/api-management-key-concepts

Now let's start by publishing a new API within Azure API management. In this case, we will using an existing publicly available API endpoint (https://conferenceapi.azurewebsites.net) as the backend application.

1. Log into the Azure Portal using the username and password provided as part of the lab module resources. Navigate to www.portal.azure.com to login.

2. In the portal search box, type "API Management Services", and *select API Management Services 

![apimsearch](https://github.com/S2FrdQ/MSBuildSecurityLabs/assets/6592423/061f9539-c37d-487f-95fd-4beacb5abab2)


3. Go to Azure API Management Portal and select *Create API Management Service*. Fill the required details and select the resource group you created and click *Review + Create*.

![createapimservice](https://github.com/S2FrdQ/MSBuildSecurityLabs/assets/6592423/16ff6ea3-392f-4611-aff5-f5215fd62820)


![createapim2](https://github.com/S2FrdQ/MSBuildSecurityLabs/assets/6592423/a0458f05-b271-4582-9513-71f018b263e3)


Make sure to select the *Developer Pricing Tier*
*Note: This should take around 30 - 40 minutes to create and activate the API Management Service.*

4. After creation of the API Management service, navigate to the left navigation pane of your API Management instance, select APIs

![66xqb7jh](https://github.com/S2FrdQ/MSBuildSecurityLabs/assets/6592423/06afe38d-48e5-467c-98d3-742e0d993394)


5. Select the OpenAPI tile.

6. In the Create from OpenAPI specification window, select Basic.

7. Enter the following values:

- OpenAPI specification --> https://conferenceapi.azurewebsites.net?format=json
- Display name --> Labs User X API (where X is your Lab User ID)
- Name -->  Once you enter the preceding Display Name, API Management fills out this field based on the JSON.
- API URL suffix --> labID_X (where X is your Lab User ID)

![newapi](https://github.com/S2FrdQ/MSBuildSecurityLabs/assets/6592423/b2236726-75df-4e81-924b-c146b142340b)


8. Select Create to create your API.

This completes the first module, for publishing a new API in Azure API Management

## MODULE 2: DISABLE AUTHENTICATION ON YOUR API ##

1. Select on the API you just created

2. Select the Settings tab and disable the option *"Subscription required"* toggle. Subscriptions are one method of enabling authentication on APIs published in Azure API management. Other methods include enforcing authentication through policies which validate JSON Web Tokens (JWT) or client certificates. By default, subcription managed authentication is enabled in Azure API Management. By disabling this toggle, your API no longer requires any authentication for access. In the remainder of this module, we will generate test API traffic to this API, and in a later module (Module 4), we will see that this API was marked as "unauthenticated" by Defender for APIs

![cwtu1q40](https://github.com/S2FrdQ/MSBuildSecurityLabs/assets/6592423/61dcfef3-2aff-4fab-8e8e-f4a75b2a8348)


3. Save to Close this view

4. Next, Navigate to the *Test* tab and select the *GetSessions* API operation

![oaavdwrg](https://github.com/S2FrdQ/MSBuildSecurityLabs/assets/6592423/f0ba0671-1081-4907-9a78-09c1c231b35f)


5. Select *Send*. This will send a request to your newly published API endpoint, this will route traffic first through the Azure API management service, and then to the backend API endpoint (https://conferenceapi.azurewebsites.net)

6. You can then scroll through the results to verify that the API has been onboarded successfully to Azure API Management, and the API has handled the request successfully (Verified by the 200 Status OK)

![oxd5pfvy](https://github.com/S2FrdQ/MSBuildSecurityLabs/assets/6592423/e267c186-0425-4093-beac-a9c23c560416)


7. Next, navigate back to the *Design* tab and select *Add Policy* under *Inbound Processing 

![policy](https://github.com/S2FrdQ/MSBuildSecurityLabs/assets/6592423/55255964-9035-494b-9c4f-88ec4c7194cd)


8. Here you can see a full set of inbound policies that can be enabled on the Azure API Management service. For Inbound Processing, these policies will be applied to all requests coming into Azure API Mangement, similar policies can be sent for outgoing responses as well, through outbound policies. Many of these include security related policies, such as enforcing rate limiting or enforcing authentication via validating jwt-token policies. For the scope of this lab, we will not be enabling any of these policies test, though please feel free to look over the available policies. More details are available here: https://learn.microsoft.com/azure/api-management/transform-api

![4gvtzfn4](https://github.com/S2FrdQ/MSBuildSecurityLabs/assets/6592423/f001d5b1-850f-46f7-a472-0d203d084d69)


You have completed this module. Please note, it may take between 20-30 minutes for the API you just published to be show up in Defender for Cloud onboarding recommendation, to allow the API to be onboarded. Once available in Defender for Cloud, you can select the API to onboard. Once onboarded in Defender for Cloud, we will be able to see various security insights for your API (including how authentication was disabled). Hang tight! We will come back to this for you to see after you complete the next module.

## MODULE 3: IaC CHECKS FOR API MISCONFIGURATIONS IN CI/CD ##

While we wait for your new API results to become visible in Defender for Cloud, let's switch gears and now focus on how to enable security in early stages of the pipeline. Azure Resource Manager (ARM) templates are a very popular form of “infrastructure as code”, a process used to programmatically create, update, and delete Azure resources. During this exercise, you will be remediating security misconfigurations embedded within an ARM deployment template.

1. Go to Azure DevOps project using this link - https://dev.azure.com/ContosoDfDBuild2023/Lab161, and login using your Azure credentials once prompted

2. Under Repos, select the LabUser_"X" where "X" corresponds to your VM Login ID. For example, if your VM login ID is 22, select LabUser_22

![la4r43l4](https://github.com/S2FrdQ/MSBuildSecurityLabs/assets/6592423/fc84dd8f-f371-4a0c-b340-aafd9ae67b70)


3. This lab will use the Microsoft Security Azure DevOps extension, specifically the infrastructure as code (IaC) template scanner. The extension has already been installed on the repository, and there is an existing pipeline workflow file (“azure-pipelines.yml”) which will run the extension security scans during for each commit to the main branch.

![etc1okml](https://github.com/S2FrdQ/MSBuildSecurityLabs/assets/6592423/989be364-964a-422a-ad10-9bcde875cc67)


*Note: No further action is required to set up the security tool. To learn more about the Microsoft Security Azure DevOps extension and the IaC scanning (powered through the template-analyzer tool), please visit --> https://learn.microsoft.com/azure/defender-for-cloud/azure-devops-extension.*

4. Once you are in the pipeline workflow file, you will see the pipeline is configured in line 19, to break (stop) the build, in the case any of the Microsoft Security DevOps find security results. This enforcement will block any insecure PR change from completing as well.

5. Next, change from the main branch to the *bases/IaC* branch, then select the “Files” menu bar item

![6xi2e7mq](https://github.com/S2FrdQ/MSBuildSecurityLabs/assets/6592423/6376af79-58c1-40e8-87b5-eba76b4244c2)


6. Within the *bases/IaC* branch, you will see that there is an Azure Resource Manager (ARM) template file, “vulnerableLab161demo.json”, used to create an Azure API management service, publish APIs, and create an Azure API Management service backend.

![a6sb85ke](https://github.com/S2FrdQ/MSBuildSecurityLabs/assets/6592423/e6591a4c-3cd5-4baf-8833-e48cdbf41888)


*Notice: This template contains API security misconfigurations which results in an increased attack surface risk and potentially exposing sensitive data if used to deploy Azure resources.

During this exercise, your goal will be to successfully merge the changes from the *basis/IaC* branch into the *main* branch, via a pull request (PR). To merge the PR, you will need to fix security issues within the ARM template by running the IaC scanning task within the pipeline to discover these misconfigurations and following the guided remediation steps to fix the issues. Once the issues are fixed, you will no longer see any template-analyzer scan security findings shown in the pipeline build, the PR build checks will pass, and your Azure API Management service template deployment is now secure.

Let's continue…

8. Start by triggering the Microsoft Security DevOps workflow step to run, by creating a new pull request for the *bases/IaC* to the *main* branch.

9. Select the “Pull requests” menu bar item

![44go66ex](https://github.com/S2FrdQ/MSBuildSecurityLabs/assets/6592423/9cb7f6ce-767d-4fe3-a830-af7727974cea)


10. Select “New Pull request”

![cpw2n3so](https://github.com/S2FrdQ/MSBuildSecurityLabs/assets/6592423/0daf5c68-e453-4eb1-a7f7-4ff639e8e40f)


- Ensure the pull request is pulling bases/IaC into main
- Select “Create”
- After selecting “Create”, you will be taken to the Pull Request Details page
- Select the “Run Defender for DevOps” step

![4tet5b6y](https://github.com/S2FrdQ/MSBuildSecurityLabs/assets/6592423/46341683-56f4-4f4e-9e42-ffb201daeed5)


*Note: it may take a few minutes for the scan to complete (you may not see “Errors” until the scan is complete)

11. Select “Job”

![8bsrvga3](https://github.com/S2FrdQ/MSBuildSecurityLabs/assets/6592423/196af7b4-9ca8-4e79-9ed0-d0813ff12e5a)


12. Select the “Microsoft Security DevOps” step

![o7rjhmew](https://github.com/S2FrdQ/MSBuildSecurityLabs/assets/6592423/f189286e-91a4-43d1-a186-ee2f9054fce2)


13. Wait for the step to complete (this may take up to 5 mins)

14. In the build output terminal, you should see the following errors (in red). These errors have also stopped the PR from completing.

![7io7qy2k](https://github.com/S2FrdQ/MSBuildSecurityLabs/assets/6592423/54d45758-8af4-465c-b530-cd99d726f08f)


15. The build output terminal contains details on which security misconfigurations were found for the ARM template and where they were found (Line X, Column X).

*You can read more about each found vulnerability and method to remediate by clicking on the related “https://github.com/Azure/template-analyzer…” link for each found misconfiguration, which will open the template-analyzer tool documentation. Scroll down in the documentation to find information about each rule. In this case, we can see that rules TA-000029, TA-000032, TA-000037 were triggered.

16. Here is an example for rule TA-000029:

![eus3i1f6](https://github.com/S2FrdQ/MSBuildSecurityLabs/assets/6592423/59ac970b-4ca1-44a6-b7ab-c21a5e9e40f3)


17. Next, let’s fix the vulnerabilities discovered in the ARM template.

18. Navigate to the ARM template file in the repository. Select the “Repos” menu bar item on the left, select the bases/Iac branch, and select the “vulnerableLab161demo.json” file.

![s4v7znkf](https://github.com/S2FrdQ/MSBuildSecurityLabs/assets/6592423/e453747b-01bf-4359-9194-085ca7249e59)


19. Select “Edit”

20. Open the Build Output Terminal browser tab and note which lines each vulnerability was found.

![mgvdcx03](https://github.com/S2FrdQ/MSBuildSecurityLabs/assets/6592423/0af5ad72-5df4-49ab-a29b-d8a07142bf15)


21. Let’s start by fixing the first reported issue, TA-000029. In this case, the Azure API management API allows for both HTTPS and HTTP traffic.

22. In the template-analyzer tool documentation (https://github.com/Azure/template-analyzer/blob/main/docs/built-in-rules.md), we can see that for rule TA-00029 it is recommended to restrict the protocols field to only accept HTTPS.

![p7ev0und](https://github.com/S2FrdQ/MSBuildSecurityLabs/assets/6592423/c6262269-c051-44fb-854e-96e32a4008a0)


23. In the “vulnerableLab161demo.json” file, navigate to line 73. Delete line 73, which will remove the “HTTP” field from the protocol’s property (ensuring only "HTTPS" traffic is allowed)

![m495yh0k](https://github.com/S2FrdQ/MSBuildSecurityLabs/assets/6592423/187c53c6-d39f-450f-8cb7-431ea616c03c)


24. Similarly, we can remediate the other found security misconfigurations as well. For rule TA-000032, API Management calls to API backends should not bypass certificate thumbprint or name validation. Let’s fix this issue by setting the *tls.validateCertificateName* and *tls.validateCertificateChain* properties to true line’s 108, and 109 of the “vulnerableLab161demo.json” file.

![atyft2te](https://github.com/S2FrdQ/MSBuildSecurityLabs/assets/6592423/c2968bf4-536a-47a7-b5e0-4651cb8418f5)


25. Lastly, for rule TA-000037, API Management subscriptions should not be scoped at the all API scope. Let’s fix this issue by setting the properties.scope property to */apis/testprotocol* in line 54.

![jgf1xwky](https://github.com/S2FrdQ/MSBuildSecurityLabs/assets/6592423/a963f554-2341-44a9-8cc8-d99532df048b)


26. Once these changes are successfully made to the file, select “Commit” in the top right corner, and select “Commit” again in the side blade.

![tn4ei0z8](https://github.com/S2FrdQ/MSBuildSecurityLabs/assets/6592423/3e8ebaa5-b10c-46a5-81c4-192e3cae6d6c)


27. This commit will start a new Pipeline build, navigate to the build. After you select “Commit”, navigate to the Pipelines page, by selecting the “Pipelines” menu-bar item

28. Select your running Pipeline

![86fj3bx9](https://github.com/S2FrdQ/MSBuildSecurityLabs/assets/6592423/06ffa935-5a3e-4d6e-bea0-eb2c24f763b4)


29. Select your commit run

![sw9jj88m](https://github.com/S2FrdQ/MSBuildSecurityLabs/assets/6592423/7ad680b4-24ad-4c9c-96d0-71a783126456)


30. Select “Job”

31. Select the “Microsoft Security DevOps” step

32. Wait for the step to complete (this may take up to 5 mins)

33. You should see the step pass

![vajybe0t](https://github.com/S2FrdQ/MSBuildSecurityLabs/assets/6592423/64002753-c385-4786-b509-3c834306086b)


34. Navigate back to the Pull Requests menu bar item under the “Repos” menu bar item. You should see all of the required checks have no passed, and this PR can safely be Completed. You have now completed this exercise and secured the Azure API Management deployment template.

![fr9tudoe](https://github.com/S2FrdQ/MSBuildSecurityLabs/assets/6592423/08fa2ef5-7a15-4ac4-b57c-122094493b6e)


## MODULE 4: VIEW SECURITY FINDINGS AND STEPS TO REMEDIATE ##

Great work! Next let's get back to viewing the API you published earlier along with the security misconfigurations associated with that API within Defender for Cloud.

1. Navigate back to the Azure API management "Conference Demo" service

2. Click the *Defender for Cloud (preview)* menu bar item on the left pane of your API Management Service portal. Defender for Cloud is a cloud-native application protection platform, providing full lifecycle security for applications. Recently Defender for Cloud added new API security capabilities, specifically providing security for any APIs published within Azure API management. Defender for Cloud has already been enabled on the subscription; however, requires per API level onboarding. In this case, we still need to complete the onboarding for your newly published API endpoint. Before we complete those steps, let's walkthrough a few example API security scenarios covered by Defender for Cloud

![xsc56xpb](https://github.com/S2FrdQ/MSBuildSecurityLabs/assets/6592423/e52df65d-f4d1-4fb4-9594-996417a20fc8)


3. You will see all the security recommendations shown here for your API Management Services and the APIs you created. In this case, we can see that even if security misconfigurations weren't caught from the IaC security checks (Module 3), they would still be scanned and found in production through these security recommendations.
![jeih52s1](https://github.com/S2FrdQ/MSBuildSecurityLabs/assets/6592423/9a0b06b7-4e4d-4215-a53e-56c1d9e7634f)


4. Select the security recommendations "API endpoints in Azure API Management should be authenticated" to view APIs published in the API management service that didn't have authentication. *Note: Your API won't show up in this list yet, as it needs to complete the onboarding to Defender for Cloud. Once onboarded, you should see your API appear here within 30 mins, since it also disabled authentication.

5. Once you click on the security recommendation, you will see the remediation steps here that you can follow to remediate this recommendation directly within your API Management Instance.

![vdd5l7b7](https://github.com/S2FrdQ/MSBuildSecurityLabs/assets/6592423/69d239d5-31f4-4095-b997-890bc2b4ceb9)


6. To complete the onboarding for your newly published API, select the recommendation "Azure API Management APIs should be onboarded to Defender for APIs. 

![poe2d3p6](https://github.com/S2FrdQ/MSBuildSecurityLabs/assets/6592423/dc4d0516-7ca7-4576-8a60-de7e9b5a6994)


7. You should see your API appear in the drop down list. Select your API and select "Fix". Now your API is completely onboarded for security coverage, and it will scanned for security misconfigurations (including the unauthenticated APIs assessment we showed earlier) and threat detections. *(Note: it takes some time for the API show in this recommendation due to the refresh interval, if your API does not show, it will show within the next 20-30 mins. No need to wait to complete this step if that is the case, the goal of this last step in the lab was to show the complete end-to-end steps to enable security for your APIs)

![v783enpq](https://github.com/S2FrdQ/MSBuildSecurityLabs/assets/6592423/8814beb3-fcaa-44b7-83d6-33b681a69859)


For more information on Defender for APIs , you can visit our documentation guide to learn more on API security recommendations and remediation steps - https://learn.microsoft.com/en-us/azure/defender-for-cloud/defender-for-apis-posture

Thank you!
