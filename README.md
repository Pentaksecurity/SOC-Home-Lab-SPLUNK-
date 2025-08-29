# SOC Home Lab SPLUNK

# 1.Setting up the lab

First thing we did was created 2 VM in my VBox.

I got a Kali Machine (192.168.20.11) and a Windows 11 (192.168.20.10)

The networking is setup only on the internal Network.

<img width="709" height="207" alt="Screen Shot 2025-08-29 at 2 07 09 PM" src="https://github.com/user-attachments/assets/668b4917-04fa-4a43-9562-7c96813aee9d" />

So these 2 can only communicate with each other for the purpose of this lab.

# 2. Setting up the exploit and monitoring

For the purpose of this lab is to show the IOCs and the monitoring with Splunk and creating Telemetery.

We are going act as an attacker (Kali) and the defender/victim (Windows) in this scenario.

# Recon

First things first is to do some recon, we are going to use Nmap for this.

<img width="1516" height="597" alt="Screen Shot 2025-08-29 at 2 11 21 PM" src="https://github.com/user-attachments/assets/4e8329b4-bd40-4a29-8a08-ccd77c42f4ca" />

We can see we got some ports open, what we were looking for is 3389 for RDP.

# Weaponization

We now will create a payload for this using msfvenom and pretend that this was a phishing email and call it "Invoice.pdf.exe"

<img width="1111" height="183" alt="Screen Shot 2025-08-29 at 2 10 53 PM" src="https://github.com/user-attachments/assets/15aae95a-76a6-44cb-b1e4-76f23076eb9d" />

# Delivery

Now we have to deliver this Payload to the victim machine (Windows), we are just going to setup a python server.

python3 -m http.server 9999
<img width="673" height="178" alt="Screen Shot 2025-08-29 at 2 12 03 PM" src="https://github.com/user-attachments/assets/a258fafa-ae2e-4a38-8550-86fe43a99b8f" />

# Exploitation

We are now going to setup a meterpreter listener we are going to use "multi/handler" for this one and set the payload the same as when we created in msfvenom:

windows/x64/meterpreter/reverse_tcp

Mind you this is super simple and any AV will detect it but for the purpose of this lab is to show telemtery and investigate with Splunk.

We disbaled Defender for this.

Once on the victim downloads the payload and executes it we got a shell.


<img width="1080" height="848" alt="Screen Shot 2025-08-29 at 2 02 48 PM" src="https://github.com/user-attachments/assets/bad57630-606b-4831-9572-b78c581eda2c" />

<img width="646" height="597" alt="Screen Shot 2025-08-29 at 2 02 59 PM" src="https://github.com/user-attachments/assets/60d7be8a-f038-41ae-a7ff-a49b5f246459" />


And we are going to run some commmands in our shell like "net user" "net localgroup" "ipconfig"

Just to look for in splunk later.

# Now switching to Windows


Now we can run netstat -anob and look at all the connections established on our Windows machine and we find our Kali box connected on the port we set 4444


<img width="1139" height="559" alt="Screen Shot 2025-08-29 at 2 22 01 PM" src="https://github.com/user-attachments/assets/54e07abf-3133-44a4-89b0-af9a43e54053" />

We can also see the Proceess ID which now we can look at that in Task Manager and see what process is running.

We look up 6648 and we see it is our "Invoice.pdf.exe".

<img width="1066" height="541" alt="Screen Shot 2025-08-29 at 2 23 21 PM" src="https://github.com/user-attachments/assets/18728f15-6f9f-4259-9a5b-1caec880de4b" />

We are not going to Kill it yet.

Now onto Splunk:


# Splunk


In splunk we have our Sysmon add on and we can look at what happened while the the connection to out Kali box was established.

We will search for our IP in the the index we created "endpoint"

<img width="1024" height="768" alt="IP" src="https://github.com/user-attachments/assets/1d06219d-4b1e-49c6-852a-fea599da6c1b" />

And instantly we see our Index.pdf.exe

Lets search for our Invoice.pdf.exe now.

<img width="1024" height="768" alt="invoicre" src="https://github.com/user-attachments/assets/6d1b4a17-c322-4626-a71e-deb0d4f4afbb" />

<img width="1024" height="768" alt="CMD" src="https://github.com/user-attachments/assets/56d764be-72e3-4ac0-803c-9c8bb199a19c" />

We can see that Invoice.pdf.exe spawn cmd.exe, well that is suspcious...


Lets expand and find the process GUID and serch for that

<img width="1024" height="768" alt="guid" src="https://github.com/user-attachments/assets/9a9389fa-b083-417f-9056-19a6bca12478" />


We searched for our process_guid and we cna filter it and create a table.

|table _timne,ParentImage,Image,CommandLine


<img width="1024" height="768" alt="Screen CMDline" src="https://github.com/user-attachments/assets/edac3b64-abe1-4a5c-9aca-e38c712addb7" />

Now we can see the telemtery that we created we can see all the commands that were ran by us in the Kali after the Invoice.pdf.exe was executed and we had access to the command line.


The purpose of this lab was to show telemetery and being able to recognize the signs of intrusion and how to identify IOCs as well as investigate the steps take by the attackers in hour system with tools liek splunk and being able to identify malicous processes and repsond to the incident, break the chiain and erradicate malicous processes.






