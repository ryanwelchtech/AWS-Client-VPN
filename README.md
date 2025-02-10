# AWS Client VPN Project

This project involves configuring an AWS Client VPN deployment to allow workstations to securely connect to an AWS VPC. The deployment consists of six stages, each implementing additional components of the architecture.

## Stages

1. [Create Directory Service](#stage-1---create-directory-service)
2. [Certificates](#stage-2---certificates)
3. [Create VPN Endpoint](#stage-3---create-vpn-endpoint)
4. [Configure VPN Endpoint & Associations](#stage-4---configure-vpn-endpoint--associations)
5. [Download, Install, and Test VPN Client](#stage-5---download-install-and-test-vpn-client)
6. [Cleanup](#stage-6---cleanup)

---

## Stage 1 - Create Directory Service

In this stage, we set up an authentication service for VPN users using AWS Directory Service.

### Steps:
1. Open the AWS Directory Service console.
2. Click `Set up Directory` and select `Simple AD`.
3. Choose `Small` as the deployment size.
4. Set up:
   - Directory DNS Name: `corp.example.com`
   - Directory NetBIOS Name: `CORP`
   - Administrator password
5. Choose a VPC and two private subnets.
6. Click `Create Directory` and wait until the status is `Active`.

---

## Stage 2 - Certificates

This stage involves generating and importing certificates into AWS ACM.

### Steps:
#### Linux/macOS:
1. Clone Easy-RSA: `git clone https://github.com/OpenVPN/easy-rsa.git`
2. Navigate to the directory: `cd easy-rsa/easyrsa3`
3. Initialize PKI: `./easyrsa init-pki`
4. Create CA: `./easyrsa build-ca nopass`
5. Create Server Certificate: `./easyrsa build-server-full corp.example.com nopass`
6. Import to AWS ACM:
   ```bash
   aws acm import-certificate --certificate fileb://pki/issued/corp.example.com.crt \
   --private-key fileb://pki/private/corp.example.com.key \
   --certificate-chain fileb://pki/ca.crt
   ```

#### Windows:
1. Open the [OpenVPN Community Downloads](https://openvpn.net/community-downloads/) page, download the Windows installer for your version of Windows, and run the installer.
2. Open the [EasyRSA releases](https://github.com/OpenVPN/easy-rsa/releases) page and download the ZIP file for your version of Windows. Extract the ZIP file and copy the EasyRSA folder to the `\Program Files\OpenVPN` folder.
3. Open the command prompt as an Administrator, navigate to the `\Program Files\OpenVPN\EasyRSA` directory, and run the following command to open the EasyRSA 3 shell:
   ```cmd
   EasyRSA-Start
   ```
4. Initialize PKI:
   ```cmd
   ./easyrsa init-pki
   ```
5. Create CA:
   ```cmd
   ./easyrsa build-ca nopass
   ```
6. Create Server Certificate:
   ```cmd
   ./easyrsa build-server-full server nopass
   ```
7. Create Client Certificate:
   ```cmd
   ./easyrsa build-client-full client1.domain.tld nopass
   ```
8. Exit EasyRSA shell:
   ```cmd
   exit
   ```
9. Import to AWS ACM:
   ```cmd
   aws acm import-certificate --certificate fileb://pki/issued/server.crt --private-key fileb://pki/private/server.key --certificate-chain fileb://pki/ca.crt --profile iamadmin-general
   ```
10. Open the AWS Console, type `ACM` or `Certificate Manager` into the search box, then right-click and open in a new tab.
11. Verify that your certificate exists in the `us-east-1` region.


---

## Stage 3 - Create VPN Endpoint

### Steps:
1. Open the AWS VPC console.
2. Go to `Create Client VPN Endpoint`.
3. Configure:
   - Name: `ClientVPN`
   - Client IPv4 CIDR: `192.168.12.0/22`
   - Authentication: `Active Directory`
   - Directory ID: Select the directory created in Stage 1
   - Server Certificate: Select the imported certificate
4. Enable split-tunnel and self-service portal.
5. Click `Create Client VPN Endpoint`.

---

## Stage 4 - Configure VPN Endpoint & Associations

### Steps:
1. Open the VPC console and go to `Client VPN Endpoints`.
2. Select the `ClientVPN` endpoint.
3. Go to the `Associations` tab and click `Associate`.
4. Select the VPC and a private subnet.
5. Wait for the status to change to `Available`.

---

## Stage 5 - Download, Install, and Test VPN Client

### Steps:
1. Download the VPN client configuration file.
2. Install the AWS VPN Client from [AWS VPN Client Download](https://aws.amazon.com/vpn/client-vpn-download/).
3. Load the configuration file into the VPN client.
4. Connect using the administrator credentials from Stage 1.
5. Verify connectivity using `ping`.
6. Add an authorization rule to allow access to the VPC.

---

## Stage 6 - Cleanup

### Steps:
1. Disconnect the VPN client.
2. Delete the VPN Endpoint.
3. Remove the Directory Service.
4. Delete the imported certificate from ACM.
5. Verify that all resources have been removed.

---

## Conclusion
By completing these steps, you successfully configured an AWS Client VPN deployment to allow secure workstation connections to an AWS VPC.
