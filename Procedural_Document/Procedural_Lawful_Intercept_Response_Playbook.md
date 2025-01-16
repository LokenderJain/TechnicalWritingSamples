# **Cover Sheet**

## Title

Lawful Intercept Response Playbook

## Type

How-to steps (procedures)

## Purpose

This document is an excerpt from a playbook written for the Alpha's Security team. It provides step-by-step instructions for responding to lawful intercept requests made by law enforcement agencies. The document includes critical processes such as gathering necessary information, setting up a VPN tunnel using Terraform or a GUI, and troubleshooting potential issues.

## Questions

*Whether the content was written solely by you, or if you were part of a team that worked on the content. If you were part of a team, please identify as exactly as possible which portions were your responsibility.*

‣ I authored over 75% of the content, gathering information through interviews, emails, and chats with Network Engineers. I obtained the code samples and screenshots from the engineering team, as I did not have access to the systems due to security restrictions. I followed the Google Developer Documentation Style Guide.

*If you used generative AI during the writing process, please describe how the tool was used.*

‣ I did not use AI to generate content for this document. However, I leveraged prompt engineering to research lawful intercept processes, including legal and compliance aspects, IP range conflicts between VPN endpoints, and how Terraform enables VPN tunnels through infrastructure-as-code principles. This ensured the documentation was accurate and accessible to less technical audiences.

# 

# **Lawful Intercept Response Playbook**

# **Introduction**

The Alpha CALEA (Communications Assistance for Law Enforcement Act) system, provided by the vendor Utimaco, allows Alpha Security to perform lawful intercepts of communications upon receiving a request from Alpha LEIS (Law Enforcement Information Security). This document is a comprehensive guide for Alpha Security team members to respond to such surveillance requests. It outlines the necessary steps, access requirements, and troubleshooting procedures involved in the process.

### **Revised Surveillance Types**

Alpha is required to deliver different information to Law Enforcement Agency (LEA) based on the type of surveillance request:

* **Title III:**  
  * RADIUS information.  
  * Complete port mirror of all ingress and egress traffic of the surveillance target.  
* **Pen Register:**  
  * RADIUS information.  
  * Packet data header reports (IP addresses, protocol, ports, and other metadata).

**Prerequisites**

To activate surveillance, you need access to the following systems:

* Alpha shell server (for example, shell.example.com)  
* Valentine entry for [CALEA Admin 2](http://..)  
* Valentine entry for [CALEA Op 1](http://..)  
* Cloudtop  
* [Utimaco GUI](http://..)  
* Prod. Access

**New Surveillance Requests Process**

To handle new surveillance requests, follow these steps:

1. Gather the following information from the LEA:  
   1. Monitoring Center (MC) IP address: The IP address to which intercepted data will be sent.  
   2. VPN connection IP Range: The remote network IP range required to establish the VPN connection.  
   3. Data Transfer Port(s): The specific port(s) used for transferring intercepted data, including CC (Call Content) for Broadband (BNG) traffic and CD (Call Data) for Signaling (RADIUS) traffic.  
   4. Surveillance Target Information: This includes customer identifying information such as name, address, and email.  
2. Use the provided target information to obtain the Alpha Subscriber ID. Refer to [Access Customer Identification](http://..) to locate the Subscriber ID. Ensure that the Subscriber ID includes the "0001" suffix (essential for accurate identification within the CALEA system).  
3. **Create VPN Tunnel (Terraform):** Terraform is an infrastructure-as-code tool used to provision and manage cloud resources. In this step, we'll use Terraform to create a secure VPN tunnel between Alpha compliance system and law enforcement.  
   1. Generate a new shared secret for the VPN tunnel on [Valentine](http://..).  
   2. Create a CL to modify the [main.tf](http://main.tf) file.  
   3. Create a new ciphertext encrypted with the alpha-xre keyring: This will be used to securely store the VPN's pre-shared key.  
      1. Run the following command on Cloudtop, replacing "PASSWORD" with a new, strong password: 

| set \+o historyecho "PASSWORD" | gcloud \--project\=alpha-secrets-c kms encrypt \--plaintext-file\=- \--ciphertext-file\=- \--keyring\=alpha-xre \--key\=xre-generic \--location\=us-central1 | base64 \--wrap\=0set \-o history |
| :---- |

      2. Troubleshooting:  
         1. If you encounter the error "ERROR: (gcloud.kms.encrypt) UNAUTHENTICATED," ensure that you are logged in to gcloud and have the necessary permissions. Run gcloud auth login to authenticate and retry the command.  
      3. Copy the output (the base64-encoded ciphertext) and store it securely. You'll need it for the next step.  
   4. Create a new [data point](http://..) in the [main.tf](http://main.tf) file to store the VPN pre-shared key.  
      1. Paste the ciphertext after the last data point in the main.tf file.

| data "alpha\_kms\_secret" "lea\_mc\_secret" { crypto\_key \= "alpha\-secrets\-c/us\-central1/alpha\-xre/xre\-generic" \# https://valentine.corp.alpha.com/\#/show/1652123884858936?tab\=metadata ciphertext \= "CiQAUOZccR5EGBCfYZlogOEI\+TSenU/J6NXsuonlANpGl\+NnbS8SNgB/TK0j7RlxU54c2gS00vYdGvGpGwwbE2cG3he/isC7k7r2sGiWGPH\+grKW8vKK2/vunm4VxQ\=="} |
| :---- |

      2. Review the variables information:  
         1. \`Data:\` This is the new variable name to store VPN pre-shared key information.  
         2. \`crypto\_key:\` This stays the same.  
         3. \`\#:\` This is a link to the Valentine password (for our record)  
         4. \`ciphertext:\` This is the output of running the command on Cloudtop in step “c”.  
      3. Replace "lea\_mc\_secret" with ”something\_new\_secret”.  
   5. Paste the code for the "lea\_mc" module at the bottom of the Terraform configuration file ([main.tf](http://main.tf)) and update the following variables to match the LEA's information:

| module "lea\_mc" { source \= "../../modules/lea\_mc" name            \= "FBI\_MC\_K12312412" vpc\_name        \= local.vpc\_name peer\_ip         \= "136.1.1.5" shared\_secret   \= data.alpha\_kms\_secret.FBI\_MC\_K12312412\_secret.plaintext routes \= {   "test-calea-lea-new" \= {     dest\_range \= "136.1.1.5/32"     priority   \= 1000   } }} |
| :---- |

      1. module**:** Keep "lea\_mc" and add the case number to the name, starting with an underscore ("\_"). For example: "**lea\_mc\_K5235234**"  
      2. source**:** Leave this value unchanged. This defines the location of the module code.  
      3. name**:** Create a descriptive name that includes the LEA name, MC, and the LEA case number. For example: "FBI\_MC\_K12312412"  
      4. vpc\_name**:** Leave this value unchanged. This refers to the VPC network where the VPN will be created.  
      5. peer\_ip**:** Enter the IP address of the LEA's VPN endpoint.  
      6. shared\_secret**:** Use the name of the data point you created in Step 4 to store the VPN's pre-shared key. The syntax should look like: \`data.alpha\_kms\_secret.YOUR\_KEY\_NAME.plaintext\`. (Replace "YOUR\_KEY\_NAME" with the actual name you chose.)  
      7. routes**:** Add a new route with the LEA's IP address (using the /32 or /30 CIDR notation provided by the LEA) and a priority of 1000\. 

         **Important:** This is the LEA's network IP address, *not* the VPN endpoint IP. 

         **Caution:** If the LEA is using an internal IP address, ensure it doesn't overlap with our internal IP range to avoid routing conflicts.

   6. **Syntax Check:** This step ensures that your Terraform configuration file is valid and ready to be applied.  
      1. Authenticate using \`gcloud auth login\`.  
      2. Authenticate application default credentials using \`gcloud auth application-default login\`.  
      3. Create a Terraform alias in your Linux environment:

terraform='/alpha/data/ro/teams/terraform/bin/terraform'

4. Initialize Terraform: Run \`terraform init\`   
   5. Generate an Execution Plan: Run \`terraform plan\`.  
   7. Send CL for review.  
   8. Submit CL.  
      1. Annealing will automatically push the configuration. **Note:** This process may take up to an hour. You can check the push status [here](http://..).  
      2. Confirm with LEA: Once the configuration is pushed, verify with the LEA that the VPN tunnels are established.  
4. **Emergency: Create VPN Tunnel (GUI)**  
   **Note:** Skip this step if you have already created a VPN tunnel using Terraform in Step 3\. Use this GUI-based method to create a VPN tunnel only in emergency situations where the Terraform method is unavailable. 

1. Log in to **[Pantheon](http://..)** and select "project-x" project.  
2. Click **"VPN SETUP WIZARD"**

   ![][image1]

3. Select **"Classic VPN."**  
   1. Configure Alpha Compute Engine VPN gateway:  
      1. **VPN Name:** Enter a descriptive name (for example, "vpn-gw-classic-lea-mc-CASE\_NUMBER").  
      2. **Description:** (Optional) Add a brief description.  
      3. **Network:** Select "project-x-vpc".  
      4. **Region:** Choose an appropriate US region (the auto-selected region is usually acceptable).  
      5. **IP address:**  
         1. Select **"Create IP address."**  
         2. **Name:** Enter a descriptive name (for example, "vpn-classic-lea-mc-CASE\_NUMBER-ip").  
   2. **Configure Tunnel:**  
      1. **Name:** Enter a descriptive name (for example, "tunnel-vpn-classic-lea-mc-CASE\_NUMBER").  
      2. **Description:** (Optional) Add a brief description.  
      3. **Remote Peer IP address:** Enter the IP address provided by the LEA.  
      4. **IKE Version:** Select "IKEv2." (If the LEA requests IKEv1, explore using IKEv2 instead.)  
      5. **IKE pre-shared key:** Generate a secure key using Valentine and share it securely with the LEA.  
      6. **Routing options:** Select "Route-based."  
      7. **Remote network IP Ranges:** Enter the IP range provided by the LEA for traffic forwarding.  
      8. **Click "Create."**  
      9. **Confirm with LEA:** Verify with the LEA that the VPN connection is established.

    


       


[image1]: <data:image/png;base64,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8eLFLif3Ds61Pn+4P0aPHm3LuD/efPNNt23ps2nTJvODH/zACZw+Bn97w4cPd30QUfdH165dA9v54IMPXH3FihU2h/3JMWM5a9Ys19/fvj4GfTxRbfo8XnrppfY8olzI98dFF11kfxmUuvw88T/rMcccY+v/8i//Yuv333+/HbFD+ZRTTnH95Pz+27/9W6TAJSoj8Auh1K+44gpXFqmU8NfB6DgFjsSSuAjcddddZ0477TSzcOFCW9cjcLJvCByEBmX5YeC3Y/n+++8HcskCfS6//PLQdqScyjayHZjq+/GPfxzKI3zJlZz/2VPJ+eH3++ijj1z5X//1X02NGjVsWaYXUV6+fHnkthEQsBkzZthyzZo17chM1L78EZao4zv11FNdHceA8mWXXRbaFu6ZV1991T50kTvuuONCffyyL3DY7n/8x38E+jz00EOBqfxCChGPqGsmffR1g4jrnI6odtkPRohRxv0qAof74+STTw6tI3W5P1DO9P7Acvv27e44zj333Mj7A0u5P37729+WeX9gqc+j36eQ7w99LXQey9tuu82WIcj4xThK4DDCevfdd9tyohE4XH8sTzzxRJu76667zNdff23LzZs3N927d7ftWuD843n88cfdz3F7n+kHHyFxIE4CJ2UEBK5bt27m3//93900Itp9gfN/I/Z/EOEfv8SaNWtC+0sWufqM6YZ/HEOHDjX/+Mc/bDnbAocf2tOmTQv180fg9Pro//Of/zy0PV86ReB++ctf2n4rV650/fUD2r9eyOGHPaY38fBFe+3atc2OHTvcfYCHCpZXX32128af//xns27dusjjlXLUCJwf2L//XlwhhR45ktDnd8qUKaGc9NPrSj7q/tD9ROBQxshY1P2htyH3x9///ve07g/kzjvvPHd/IOT+wC+Bcn/4+0KUdX9gmeg8Igr5/rjwwgsDI3Dy80Q+K5a9evWyZcgrfjmOErjnn3/e9OnTx5YTCRyWGNWWX7780TuM+OJ6o18ygcN19K87BY7EkrgJHMQNdUydXHnllfY9FH+koCyBg0RgKkCmifS+yopM1slF4DhOP/10O4WF8ksvvWTzvsDhhyF+mPojHs2aNbPnTT4Hlp988ol55plnEn425PGOj19PJnBS1tubN2+efXjLtBemyLDEb9D9+vWzZYxg4Hpi1ARTX08//bQtY3oM103v609/+pMrt2jRwpYheFhieivRsUgeDxkplyVwON86VyiRSDzw7wHvI/n/HrDs3Lmzna6UaciodRGPPfaYbdP3h+7nC5z00f0wAhZ1f/Tu3dvdH+gn94dsB+Xx48ebM844w+beeust11ffHxdffLG7P/xjkX8jfug+WCY6j4hCvj8Q+Fz4WdCqVSv3GfXnXrZsmV3iGmC07cwzzwxMUcs6mFaG5OEVBL0PXcYXiU444QS7juQwpYvyqFGj7BK/rMs6GKXFOvh5gPIFF1xAgSPxJA4Cl09RFT5jRYQvY7kOTBfec889oTwineuJaSOdY2QvIHlSTue6lDeSfcNY3sssKyAYvD8qLyhwJJZQ4LIbVeEz5jL+9re/2Wlv+aJDruPXv/515GihH506dQrldGCEQOcY2Q+5P3Q+V1HW/YHXE1K5Px5++OFQjlFxQYEjsSQbApdqXH/99aH1GQwGg8HIZVDgSCwpr8AxGAwGg5HPQYEjsYQCx2AwGIw4BwWOxBIKHIPBYDDiHBQ4EksocAwGg8GIc1DgSCyBwBFCCCFx5f8DybPSZ46trYAAAAAASUVORK5CYII=>