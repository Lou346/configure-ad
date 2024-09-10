p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

<h1>On-premises Active Directory Deployed in the Cloud (Azure)</h1>
This tutorial outlines the implementation of on-premises Active Directory within Azure Virtual Machines.<br />






<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Active Directory Domain Services
- PowerShell

<h2>Operating Systems Used </h2>

- Windows Server 2022
- Windows 10 (21H2)

<h2>High-Level Deployment and Configuration Steps</h2>

- Domain Controller VM  named “DC-1”
- Domain Controller’s NIC Private IP address to be static
- ICMPv4 (ping) was allowed on the Domain Controller
- Create an Admin and Normal User Account in Active Directory
- Join Client to domain
- Attempt to login Client-1 with one of the users

<h2>Deployment and Configuration Steps</h2>

<p>Firstly, we will need to establish the resource group so that you can add your virtual machines for the Domain Controller (DC-1) and the Client Virtual Machine (Client-1). The Domain Controller VM will use a Windows Server 2022 system image (a serialized copy of the entire state of a computer system stored in some non-volatile form such as a file).

</p>
<p>

  
  
  
  
  
  ![image](https://github.com/user-attachments/assets/8a8d36a1-f6ba-424c-b205-5786d4fcf8e2)




</p>
<br />

<p>


  
  
  

</p>
<p>
The Client VM (Windows 10) named “Client-1” was created with the same Resource Group and Vnet that was created in DC-1.
</p>
<br />

![image](https://github.com/user-attachments/assets/9b8a9372-33cc-475c-b7b8-ebef1488394d)

</p>
<p>
.Private IP address set to static (static IP addresses are necessary for devices that need constant access.)



![image](https://github.com/user-attachments/assets/075c5bf6-1fac-4a45-835a-b7f6fcde2017)

  
</p>Second, check for a connection between the client device and domain controller by logging into Client-1 with Remote Desktop Connection (RDP) and pinging DC-1’s private IP address using ping -t (perpetual ping). ICMPv4 (ping) was allowed on the Domain Controller's (DC-1) Firewall in Windows Firewall (Core Networking Diagnostics - ICMP Echo Request (ICMPv4-In)). After logging back into Client-1 check to make sure the ping is successful.


![image](https://github.com/user-attachments/assets/8d23df5e-1742-43ca-9446-11800866c75b)

<br />



Pictured below displays that the ICMP rule has been allowed on the Windows firewall for inbound traffic:




![image](https://github.com/user-attachments/assets/c123b2e5-6aa8-4d99-9d1c-641c31fb59b5)





While in DC-1, we've selected to 'add roles and features' to enable Active Directory Domain Services. Promoted as a Domain Controller (DC): a new forest as mydomain.com setup. Remote Desktop was Restarted and logged back into DC-1 as user: mydomain.com\labuser.




![image](https://github.com/user-attachments/assets/d29d3ac1-e138-402c-995c-2e70ebefd27f)





![image](https://github.com/user-attachments/assets/4bff9f03-ed26-4bd7-a5e5-37ed69674c6c)




Next, we configure the organizational units for the admins and employees in Active Directory (AD) while continuing to be in DC-1 (Remote Desktop Connection). The accounts can now be viewed in Active Directory in the appropriate organizational unit. In the Active Directory, right-click on your domain name and move your mouse to hover new-->Organizational Unit and left-click to create folders for your AD. We will create employees, admins, and security groups.




![image](https://github.com/user-attachments/assets/4783b0d2-1a2c-4a60-b2d8-f1b609b7e4c3)






Create a new OU named '_ADMINS' --> Create a new employee named Karen What (same password) with the username of 'karen_admin'. Once the admin is created, add "karen_admin" to the "domain admins" security group.





![image](https://github.com/user-attachments/assets/7aeff918-6c4f-4a48-960d-0b64692a1731)




Log out and close the connection to dc-1 for the current user(mydomain.com\labuser) and log back in as "mydomain.com\karen_admin".





![image](https://github.com/user-attachments/assets/b4c91602-d2ba-4565-bce6-2755e79e2b2c)





Next, we'll join Client-1 to the domain (mydomain.com); however, we must change the DNS on Client-1 to the private IP address of DC-1 so that we can properly add client-1 to the domain. Here we will select the NIC on client-1 to change the DNS to the private IP address of DC-1






![image](https://github.com/user-attachments/assets/eb0b007a-491f-4861-bba7-a74661b0d242)






Select 'DNS Servers'




![image](https://github.com/user-attachments/assets/4853f38f-8d6f-42f9-8dd3-871472b38ec0)





Select the 'Custom' radio button for DNS server so that you can now enter the DC-1 private IP address.




![image](https://github.com/user-attachments/assets/9b0c6e26-2b56-401a-b045-50b9bc0635eb)



Now that we have successfully changed the DNS server to the private IP address of DC-1, we can add client-1 to the domain without error. You will receive a message letting you know that the client has been successfully added to the domain. This can be done by going to System > Rename This PC > enter domain name > select OK > select Apply. The update then requires a system restart.







![image](https://github.com/user-attachments/assets/696a8f3f-8b37-4767-b12a-81e3befbca37)






A message displays that the client has been successfully added to the domain





![image](https://github.com/user-attachments/assets/c90758f2-b201-40c2-821a-b07277f7a427)




Now, we can create our users that will be loaded into our _EMPLOYEES OU in the domain controller (DC-1). To create these employees we will run PowerShell_ISE as an administrator. A new File will be created then we can enter the pre-configured script into the file. When the script is run, random employees will be created






powershell
# ----- Edit these Variables for your own Use Case ----- #
$PASSWORD_FOR_USERS   = "Password1"
$NUMBER_OF_ACCOUNTS_TO_CREATE = 10000
# ------------------------------------------------------ #

Function generate-random-name() {
    $consonants = @('b','c','d','f','g','h','j','k','l','m','n','p','q','r','s','t','v','w','x','z')
    $vowels = @('a','e','i','o','u','y')
    $nameLength = Get-Random -Minimum 3 -Maximum 7
    $count = 0
    $name = ""

    while ($count -lt $nameLength) {
        if ($($count % 2) -eq 0) {
            $name += $consonants[$(Get-Random -Minimum 0 -Maximum $($consonants.Count - 1))]
        }
        else {
            $name += $vowels[$(Get-Random -Minimum 0 -Maximum $($vowels.Count - 1))]
        }
        $count++
    }

    return $name

}

$count = 1
while ($count -lt $NUMBER_OF_ACCOUNTS_TO_CREATE) {
    $fisrtName = generate-random-name
    $lastName = generate-random-name
    $username = $fisrtName + '.' + $lastName
    $password = ConvertTo-SecureString $PASSWORD_FOR_USERS -AsPlainText -Force

    Write-Host "Creating user: $($username)" -BackgroundColor Black -ForegroundColor Cyan
    
    New-AdUser -AccountPassword $password `
               -GivenName $firstName `
               -Surname $lastName `
               -DisplayName $username `
               -Name $username `
               -EmployeeID $username `
               -PasswordNeverExpires $true `
               -Path "ou=_EMPLOYEES,$(([ADSI]`"").distinguishedName)" `
               -Enabled $true
    $count++
}




Here is the script loaded into powershell prior to running the script to create 1000 random users
  <p align="center">
  <img src="https://i.imgur.com/ez4THWm.png" height="80%" width="80%" alt="powershell with script loaded"/>
  </p>
  </br>
  Random users are created now after choosing to execute the code. Here we can now see the script loading the 1000 users:
<p align="center">    
  <img src="https://i.imgur.com/f2vKx8Y.png" height="80%" width="80%" alt="powershell execute code"/> </p>
  Those random Users are now reflected in Active Directory on the Domain Controller
  <p align="center">
  <img src="https://i.imgur.com/lHBM2nh.png" height="80%" width="80%" alt="active directory shows created users"/>
  </p>
  Attempt to login on Client-1 with a random user that has been created
  <p align="center">
  <img src="https://i.imgur.com/HFguOhB.png" height="80%" width="80%" alt="windows start menu shows login user"/>



