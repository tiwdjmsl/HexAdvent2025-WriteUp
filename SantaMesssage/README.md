# 🎅 Santa's Message

| Info | Details |
|-----|--------|
| Category | Forensics |
| Author | kestryix |
| Difficulty | Medium |

Challenge Description

The challenge provides a packet capture (.pcapng) file containing network traffic.
Our objective is to analyze the capture and recover the hidden message transmitted through the SMB protocol.
Step 1 – Opening the Packet Capture

The first step is to open the provided capture file in Wireshark.

To focus only on SMB communication, apply the following filter:

smb2

This filter displays all SMB packets exchanged between the client and server.

During inspection, we observe an SMB authentication process, which uses NetNTLMv2 authentication.

Step 2 – Extracting the NetNTLMv2 Hash

Within the SMB authentication packets, the NetNTLMv2 challenge-response hash can be extracted.

In Wireshark:

Locate the SMB2 Session Setup Request

Expand the following fields:

SMB2
 → Session Setup Request
 → Security Blob
 → NTLMSSP

Inside this section we can find the NTLMv2 response values, which form the authentication hash.

The extracted hash is saved into a file called:

hash.txt

Example structure:

USERNAME::DOMAIN:SERVER_CHALLENGE:NTLMV2_RESPONSE
Step 3 – Cracking the Hash Using Hashcat

Since the authentication uses NetNTLMv2, we can attempt to recover the password using Hashcat.

Hashcat supports NetNTLMv2 using mode 5600.

Run the following command:

hashcat -m 5600 hash.txt /usr/share/wordlists/rockyou.txt
Command Explanation
Parameter	Description
-m 5600	Hash mode for NetNTLMv2
hash.txt	File containing the captured authentication hash
rockyou.txt	Wordlist used for password cracking

Hashcat performs a dictionary attack using the rockyou wordlist.

Step 4 – Password Recovered

After running Hashcat, the password was successfully cracked.

Hashcat output:

Status...........: Cracked
Hash.Mode........: 5600 (NetNTLMv2)
Recovered........: 1/1 (100.00%)

Recovered password:

alliwantforchristmasisyou
Step 5 – Decrypting SMB Traffic

Now that the password is known, Wireshark can decrypt the SMB session.

Navigate to:

Edit → Preferences → Protocols → SMB2

Enable the option:

Attempt to decrypt SMB2 data

Then add the credentials under NTLMSSP Authentication:
<img width="2560" height="1440" alt="image" src="https://github.com/user-attachments/assets/562d4696-e5e9-470d-aa6e-92f73dc790d5" />

Username: PEPPER MINSTIX
Password: alliwantforchristmasisyou

After applying these settings, Wireshark decrypts the SMB packets.

Step 6 – Locating the Transferred File

Once the traffic is decrypted, we search for files transferred via SMB.

The following packet indicates access to the flag file:

Create Request File: flag.txt

To extract the file, navigate to:

File → Export Objects → SMB

Wireshark lists all files transferred during the SMB session.
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/d68f9876-274c-4bd5-a259-a779f5b77a8e" />

From the list, locate:

flag.txt

Save the file locally.

Step 7 – Recovering the Flag

Opening the exported file reveals the hidden message:
<img width="1439" height="790" alt="image" src="https://github.com/user-attachments/assets/9e3f8314-b279-4317-9def-436c67eb88b3" />

HEX{31Ve5_uS3_1Ns3cUR3_P@$$w0rd5}

## Flag
HEX{31Ve5_uS3_1Ns3cUR3_P@$$w0rd5}
