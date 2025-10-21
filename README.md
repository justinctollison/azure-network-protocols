<p align="center">
<img src="https://i.imgur.com/Ua7udoS.png" alt="Traffic Examination"/>
</p>

# Network Security Groups (NSGs) and Inspecting Traffic Between Azure Virtual Machines

In this tutorial, we observe various network traffic to and from Azure Virtual Machines with **Wireshark** as well as experiment with **Network Security Groups**.  
<br>

## Video Demonstration

- [YouTube: Azure Virtual Machines, Wireshark, and Network Security Groups](https://www.youtube.com/watch?v=1y86ONhkGJ4)

## Environments and Technologies Used

- Microsoft Azure (Virtual Machines/Compute)  
- Remote Desktop  
- Various Command-Line Tools  
- Various Network Protocols (SSH, RDP, DNS, HTTP/S, ICMP)  
- Wireshark (Protocol Analyzer)  

## Operating Systems Used

- Windows 10 (21H2)  
- Ubuntu Server 20.04  

## High-Level Steps

- Set up two Virtual Machines in Azure, one Windows and one Linux  
- RDP into the Windows VM  
- Download and install Wireshark  
- Observe different Networking Protocols from ICMP, DHCP, DNS, and SSH  

## Actions and Observations

The very first step is we'll want to create two different Virtual Machines within the same Virtual Network in Azure. This can be accomplished by creating a Resource Group to house both Virtual Machines and then making sure both VMs are inside the same Virtual Machine when you set up their networking portion. You'll want each VM to have at least 2-4 CPUs and one to run Windows 10 Pro and the other to run Linux (for the sake of SSH). In my case I use Standard_D2s_v3. We'll be using these Virtual Machines to test common Networking Protocols and to get an idea about the packets being sent through Wireshark. After both your VMs are deployed, you'll want to use Remote Desktop Protocol (RDP) and sign into the Windows VM. Once inside, we're going to head over to **wireshark.org** and download Wireshark, make sure you get the Windows x64 Installer.

<br>
<p align="center">
<img width="2118" height="490" alt="image" src="https://github.com/user-attachments/assets/a1ca625a-5ae5-42b4-bed7-e5dd3ec552c9" />
<img width="2531" height="1367" alt="image" src="https://github.com/user-attachments/assets/d7eca98b-a08c-4afa-a8c8-07c592919f67" />
</p>

Install Wireshark and start it up. Make sure you're capturing the option that has Addresses. Usually this is either Ethernet 2 or Ethernet, you can hover over each option to double-check. It'll also list the IP Address of the current Virtual Machine you're on, in our case it is 10.0.0.4 which is our Private IP Address on our Virtual network.  

<br>
<p align="center">
<img width="999" height="771" alt="image" src="https://github.com/user-attachments/assets/7197d5f8-d40c-4559-b1bc-58296814b250" />
</p>

After starting the Capture, you'll see a lot of spam happening. These are all the packets being sent back and forth from a source and destination and using a networking protocol. We'll need to filter this out and then we can start testing specific networking protocols in Powershell. First, let's filter out the traffic with entering ICMP into the bar. It should empty the list and show nothing, as nothing has been sent yet. We'll be using "ping" in Powershell and observing the results. Minimize your RDP and go find the information for your Linux Virtual Machine. Find the IP Address for it, for us it's 10.0.0.5 and then we'll see how two devices on the same network find each other. Next head back into your Windows VM and open up Powershell and then enter "ping 10.0.0.5" and then observe the results in Wireshark. You'll see 8 entries in Wireshark, while you see 4 in the Powershell. This is because Wireshark shows both the request and reply, while Powershell just shows the reply. Optionally, look through the data packets inside the Internet Control Message Protocol to see what's being sent back and forth.

<br>
<p align="center">
<img width="2398" height="1035" alt="image" src="https://github.com/user-attachments/assets/8da8cdb7-ef7e-44a3-aaef-b853c7f18cd4" />
<img width="1618" height="420" alt="image" src="https://github.com/user-attachments/assets/114247eb-fb67-469e-9a49-e6848c4a4064" />
</p>

Now that we've seen the ping command functioning, what happens when we decide to block it? Let's continually ping the Linux machine with running "ping -t" and this should continually ping the machine until we decide to stop it. This can be observed in both Powershell and Wireshark. Minimize your Windows VM and go back into your Linux VM information and then head to **Networking → Network Settings**. In here we'll create what's called a **Network Security Group (NSG)** which is also a firewall. We'll be denying access to the ICMPv4 and setting its priority just above the SSH priority. ICMP also does not utilize ports, so we can leave that alone. Head back into your Windows VM and you should see that our ping command is now timing out, you'll see a Request Timed Out in Powershell as well as only a request ping in Wireshark. After observing this, we can see the NSG working and now we can stop the continuous ping with **Ctrl+C** and also head back into our Linux NSGs and remove the security rule.

<br>
<p align="center">
<img width="2086" height="1147" alt="image" src="https://github.com/user-attachments/assets/1b610386-f244-484d-a60d-8b0881385818" />
<img width="1653" height="405" alt="image" src="https://github.com/user-attachments/assets/f85e0b0d-3b90-48da-8814-247b11b4a5b3" />
</p>

Next command we will use is **SSH or Secure Shell**. We'll SSH into our Linux Machine or Ubuntu. This allows us to log into our Linux machine through the command line purely. We can do this by entering "ssh labuser@10.0.0.5" in our case. And then inside Wireshark we will filter out the traffic by SSH. Inside Powershell, it'll show info for requesting a password and show a key exchange. Once we're inside the Linux, you'll see the Powershell username change to labuser@linux-vm and also change colors to green as well. In here we can enter various Linux commands to show more information such as "hostname" "pwd" "uname -a". And we can also create a file on that machine. Now if we observe the results in Wireshark and look through the data packets, in contrast to ICMP, the data packets are encrypted. Where we could view what was being sent in ICMP, we are not able to see it in SSH. We can also filter Wireshark through TCP port 22 and it will show the same information as SSH uses TCP port 22.

<br>
<p align="center">
<img width="1139" height="931" alt="image" src="https://github.com/user-attachments/assets/88b9c54f-aa10-484a-8298-473020cac5ff" />
<img width="805" height="425" alt="image" src="https://github.com/user-attachments/assets/701e3ae4-4d84-4415-acea-1aae66b5b581" />
</p>

Now let's filter out Wireshark with **DHCP**. DHCP or Dynamic Host Configuration Protocol assigns IP Addresses to devices on the Network. In our case, we'll be using the Azure DHCP server to assign IP Addresses to our Virtual Network. If we enter "ipconfig /renew" we will see our new IPv4 Address, however it may be the same IP Address as before. This just means we released 10.0.0.4 and the DHCP server assigned it back to us. To verify this, we can see in Wireshark where we sent a request to 168.63.129.16 which is the Azure DHCP server and then we got back an ACK message which just means an acknowledge, meaning it went through. DHCP also uses ports UDP 67 and UDP 68, we can also filter out the results through that. Optionally, we can run a script to see both a release and renew command to see in Wireshark that we release our IP Address, get set to 0.0.0.0, then send a request to the DHCP server and then we renew.

<br>
<p align="center">
<img width="1011" height="250" alt="image" src="https://github.com/user-attachments/assets/92865a38-4467-408d-8766-e91a1400cd97" />
<img width="1638" height="49" alt="image" src="https://github.com/user-attachments/assets/306793e5-8f1c-465e-b194-d3035ae9ca98" />
</p>

Our next protocol will be **DNS or Domain Name System**. DNS takes human-readable web addresses and converts them into an IP Address, so that your computer or device is able to read it. Let's filter out Wireshark to show DNS only and then go into Powershell. We will use "nslookup" and look up disney.com. In our results should be an IP Address. We can go further and take this IP Address and enter it into our web browser to see that Disney has a custom page for it, even though it sends us to an Error page 404. DNS also uses ports UDP 53 and TCP 53.

<br>
<p align="center">
<img width="557" height="158" alt="image" src="https://github.com/user-attachments/assets/cd66370f-4f63-4c4c-ba4d-dd1d1629ec2d" />
<img width="1996" height="1013" alt="image" src="https://github.com/user-attachments/assets/7a252054-7317-4123-9875-cc5242e7be4f" />
</p>

For our last Networking Protocol, we will filter out using **RDP**. RDP is Remote Desktop Protocol which is what we're currently using with our Virtual machine. If you filter out with TCP port 3389, you'll see a whole lot of spam. This is because the Virtual Machine is constantly sending data back and forth between our actual desktop and the VM. It's literally streaming video and all our inputs. This can be observed when we type rapidly or move the mouse a lot.

<br>
<p align="center">
<img width="1636" height="558" alt="image" src="https://github.com/user-attachments/assets/f902eb45-10e3-4fcc-b18f-93bb9916df52" />
</p>

So that was a quick run-down on setting up two virtual machines, exploring Networking Protocols, using Wireshark to observe data packets and more information and even using Network Security Groups to block messages. I hope this was informative and helpful!

<br>
