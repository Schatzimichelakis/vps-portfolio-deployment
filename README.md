# vps-portfolio-deployment
A guide on securing and deploying a personal portfolio on a Linux VPS using Nginx and Certbot.

## Phase 1: VPS Configuration & Nginx install
A friend of mine working in tech suggested that setting up my own VPS server would be a great hands-on project and a solid way to host my portfolio. The idea sounded both fun and practical, so after doing some basic research, I decided to give it a shot.
This is Phase 1 of documenting my setup process. It covers the initial steps I took from creating the server and hardening its security, to installing Nginx and serving my portfolio page.

<img width="1238" height="681" alt="image" src="https://github.com/user-attachments/assets/2eaac585-4045-496f-a367-9013f82e55ac" />

After conducting a small research, I decided to deploy my VPS infrastructure on VULTR, selecting Amsterdam as the server location and Ubuntu as the operating system.

<img width="1233" height="398" alt="image" src="https://github.com/user-attachments/assets/5bea138c-34d3-4fbe-adc8-22cf1332e55f" />

Upon configuring the server hostname, I initiated the deployment process. Within two minutes, the instance was fully operational. I then retrieved my root credentials to establish the initial connection.

<img width="827" height="251" alt="image" src="https://github.com/user-attachments/assets/c452ed2c-bc31-4978-8484-a4c3f492f829" />

Following the initial login, I updated the system packages to the latest versions and created a new administrative user with sudo privileges.

<img width="502" height="27" alt="image" src="https://github.com/user-attachments/assets/a359bf00-9c0a-4fef-8fed-4c0e28ad963c" />


<img width="553" height="97" alt="image" src="https://github.com/user-attachments/assets/0c519e47-07ae-4387-bd4f-cc43a9a26387" />


<img width="445" height="31" alt="image" src="https://github.com/user-attachments/assets/26e9e35e-1552-48f6-9dd3-a720c059cc05" />


Next, I proceeded with the UFW configuration, allowing SSH, HTTP, and HTTPS traffic before activating the firewall.
<img width="714" height="534" alt="image" src="https://github.com/user-attachments/assets/805c6292-7bdf-4bb5-bcd0-219f9a5b9769" />

To enhance system security, I modified the SSH configuration using nano to disable remote root login, restarted the SSH service, and successfully verified authentication using the new administrative account.
<img width="556" height="62" alt="image" src="https://github.com/user-attachments/assets/7a0b8b1a-f574-4329-88f0-b17ee53f21fa" />

<img width="158" height="21" alt="image" src="https://github.com/user-attachments/assets/cf84a3b7-1716-4fb6-9d73-722b2c7d5678" />

Finally, I installed the Nginx web server and deployed a simple HTML document to validate the setup and verify its public accessibility.
<img width="483" height="24" alt="image" src="https://github.com/user-attachments/assets/41250aa0-1870-4d54-8904-247dccb6ad17" />

<img width="544" height="234" alt="image" src="https://github.com/user-attachments/assets/9bad9d21-32e9-4edf-9adc-5711c671bccf" />

<img width="486" height="365" alt="image" src="https://github.com/user-attachments/assets/10a369c7-5e47-46f2-b245-68698be66969" />

At first, I downloaded the Remote - SSH extension and connected to my server.
<img width="827" height="191" alt="image" src="https://github.com/user-attachments/assets/5b1ee6c4-2b01-4c4a-afc3-0972a57bb67a" />

Then, I got to work to make my portfolio look alive. I updated my `index.html` file and created a `style.css` file. After spending some time troubleshooting why my custom font wouldn't render, I finally fixed it, and this is the result at the time of writing this.
<img width="811" height="728" alt="image" src="https://github.com/user-attachments/assets/5a2fa636-eb87-4d32-a788-982d28f28fbe" />


## Phase 2: Web Server Configuration & SSL Setup
Afterwards, I wanted to create a read-only account, so I created a user named `demo`. This account is not added to the sudoers group, ensuring restricted, read-only access to the server.
<img width="424" height="24" alt="image" src="https://github.com/user-attachments/assets/a0261aac-a4a9-46f8-959e-b17fca8978a9" />

Next, I created the directory for the new user. I modified the permissions so that only the owner has read, write, and execute rights to this directory.
<img width="567" height="36" alt="image" src="https://github.com/user-attachments/assets/24f55eb8-a63b-4b12-92f3-825f9e9001a2" />

I set it up without password authentication, instead a secure key pair is generated.
<img width="786" height="18" alt="image" src="https://github.com/user-attachments/assets/1fa75127-8237-42c8-a7a9-6b99d527c98b" />

Finally, I copied the public key to the `authorized_keys` file to grant access, securing it with strict permissions. The last command outputs the private key to the terminal, which is then saved locally to be shared securely with authorized visitors.
<img width="838" height="56" alt="image" src="https://github.com/user-attachments/assets/1dc6fd08-cfab-4308-9779-965aae17ef6d" />

At first, the SSH connection to the `demo` account failed, falling back to a password prompt. Initially, this appeared to be a standard file permission issue on the server side (`StrictModes` in SSH). However, after verifying the strict `chmod` and `chown` settings on the `/home/demo/.ssh` directory, the issue persisted. By inspecting the server's security logs (`/var/log/auth.log`), I discovered that the server was silently rejecting the key. This indicated a key mismatch, likely caused by a formatting or copy-paste error when initially transferring the public key to the server.
<img width="1249" height="392" alt="image" src="https://github.com/user-attachments/assets/d66856de-ec8a-4516-9db3-32985aab51fb" />

So Ι generated another fresh key directly on the server for the demo user and transferred the private key to my local Windows machine. During this process, I encountered a classic Windows filesystem quirk. When saving the private key via Notepad, Windows silently appended a hidden `.txt` extension, creating a `demo.pem.txt` file alongside the old key. This caused the SSH client to throw an `invalid format` error, as it was either reading the wrong file or failing to parse the hidden extension properly.
<img width="1458" height="530" alt="image" src="https://github.com/user-attachments/assets/1757a610-6e30-42c8-9832-e129af1fb89f" />

To resolve the local issues, I used PowerShell to force-delete the old, locked key and correctly rename the new file to exactly `demo.pem`. Since the OpenSSH client on Windows requires strict file ownership, I applied the necessary read-only permissions using `icacls`, granting exclusive access to my local user. With the permissions secured and the key matching the server's `authorized_keys` file, the SSH connection authenticated without a password prompt.
<img width="753" height="919" alt="image" src="https://github.com/user-attachments/assets/bbe4f24d-bce1-401a-a1bc-ec6b9b2928f7" />

When I decided to register a domain name, I began by researching various registrars, exploring DNS configurations, and weighing the pros and cons of using a .com versus a .gr extension. Ultimately, I chose to register a .com domain through Papaki.
<img width="737" height="696" alt="image" src="https://github.com/user-attachments/assets/ce6ec716-be25-4586-989e-deb1b7c980ff" />

After acquiring the domain, I had to configure the DNS settings. This proved relatively straightforward, as Papaki's default configuration covered most of the basics. All I had to do was update the 'A' record to point to my server's IP address and set up a 'www' redirect.
<img width="780" height="538" alt="image" src="https://github.com/user-attachments/assets/b0259129-1bdc-4825-8eaa-0f7183cf351e" />

While waiting for the DNS changes to propagate globally, I logged into my VPS with administrative privileges and created a new Nginx server block. Afterward, I removed the default Nginx site and configured the server to host my web files.

<img width="596" height="359" alt="image" src="https://github.com/user-attachments/assets/e2639af6-3cf6-42ae-8d39-2c211c7401f1" />

<img width="1031" height="229" alt="image" src="https://github.com/user-attachments/assets/9a1571e4-84ab-4548-8c04-a32db39f0b5b" />

Later, when I tried visiting my URL, I was still being redirected to Papaki's default page. Confused, I asked a friend to test the connection, and he successfully accessed my new site. As it turned out, the issue was merely local caching, and all I needed to do was perform a simple DNS flush.
<img width="271" height="33" alt="image" src="https://github.com/user-attachments/assets/ebd1a18d-f292-4bca-8ed5-c27fb3a04bd7" />

For the final part of this project, I implemented an SSL setup to ensure secure connections. The process was straightforward, beginning with the installation of Certbot.

<img width="629" height="51" alt="image" src="https://github.com/user-attachments/assets/4c029c59-3758-4887-9453-c591b51de2de" />

Next, I executed Certbot and configured it to target my specific domain names.

<img width="813" height="23" alt="image" src="https://github.com/user-attachments/assets/85bcf6e5-8958-4a6d-9a25-f139508be64f" />

After providing the necessary setup details, such as an administrative email, the SSL certificate was successfully issued. The Nginx configuration was automatically updated, resulting in a fully secure, HTTPS-enabled website.

<img width="206" height="43" alt="image" src="https://github.com/user-attachments/assets/d6f43454-a9c3-4660-be51-29967e1de88a" />


## Ending Notes
This project was a lot of fun and prompted me to learn new things, such as getting more comfortable with Linux-based systems, setting up a VPS, and deploying a web server. This will be an ongoing project, as I plan to regularly update my portfolio with new work. Stay tuned for my upcoming projects hosted on this server!

You can view the deployed portfolio here: https://schatzimichelakis.com/

**Tech Stack & Deployment Tools:**

- **Frontend:** HTML, CSS
    
- **Infrastructure:** Linux VPS ([Vultr](https://www.vultr.com/))
    
- **Web Server:** [Nginx](https://nginx.org/)
    
- **Domain Registrar:** [Papaki](https://www.papaki.com/)
    
- **Encryption:** [Certbot](https://certbot.eff.org/)
