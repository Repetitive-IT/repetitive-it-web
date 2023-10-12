---
title: "Systemd – Configure an SSH reverse tunnel"
type: post
date: 2023-10-10T18:20:14+02:00
draft: false
---
At Repetitive IT we often use hybrid technologies to get POC out of the door and these resources are often a mix of Cloud/SaaS/Old Laptops/Raspberry PIs.

To get all of them in communication and to make all the automations work properly, we make abbundant use of reverse ssh tunnel connection locally as well as remotely.

Before Systemd we used and appreciated [Djb Daemontools][1] to keep the services running and restarting in case of malfunction, at today we integrate all in systemd with an easy to use script.

Before going forward you may need to verify that you have all the firewall rules in place and the correct private key to connect to your server, this is not part of this article but you may find some help here **<a rel="noreferrer noopener" href="https://repetitive.it/connect-to-an-ssh-server-without-password/" target="_blank">connect-to-an-ssh-server-without-password</a>.**

Once you are able to connect from your client to the remote server without using the password, you can then set up a new service in systemd to restart the connection in case it will drop for any given reason, and to make sure it is started at boot time.

Steps:

**1 &#8211;** Create a service for each reverse tunnel you want to set up, and give it a meaningful name. 

<pre class="wp-block-code"><code># touch /etc/systemd/system/reversetunnel.servername.service</code></pre>

This is a &#8216;Systemd Object&#8217;, a service in this case, as it will be used to start/stop/restart/reload a specific daemon or service like Apache or like in this case an **ssh reverse tunnel**.  
We also need to make sure, Systemd understands the nature of the object and that&#8217;s why it is important to append &#8216;.service&#8217; at the end of the name

**2 &#8211;** open the file with your favourite text editor and copy paste the following

<pre class="wp-block-code"><code>&#91;Unit]
Description=Describe what this service is for
After=network.target

&#91;Service]
Type=simple
ExecStart=/usr/bin/ssh -g -N -T -o VerifyHostKeyDNS=no  -o "ServerAliveInterval 10" -o StrictHostKeyChecking=no -o "ExitOnForwardFailure yes" -R 8090:localhost:8080 user@remoteserver -i /full/path/to/your/private/key
Restart=always
RestartSec=5s

&#91;Install]
WantedBy=default.target</code></pre>

Let&#8217;s also explain it a bit

## [Unit] Section {.wp-block-heading}

**Description=Describe what this service is for**  
This line is to describe the service in a few words like Reverse SSH to server XWZ.com

**After=network.target**  
This it is kind of a dependency, it means &#8216;start this service once the network.target is up and running and just to give you a complete piece of information a **Target** for Systemd is a group of resources



## [Service] Section {.wp-block-heading}

**Type=simple** 

This is an indication for Systemd to consider the process successfully started once the main process is forked off 

**  
ExecStart=/usr/bin/ssh -g -N -T -o VerifyHostKeyDNS=no -o &#8220;ServerAliveInterval 10&#8221; -o StrictHostKeyChecking=no -o &#8220;ExitOnForwardFailure yes&#8221; -R 8090:localhost:8080 user@remoteserver -i /full/path/to/your/private/key**

This is the actual command to start the reverse proxy, we are executing an ssh client against a server so let&#8217;s check out what are the options used on our command:

  * **/usr/bin/ssh** is our executable
  * **-g** Allows remote hosts to connect to local forwarded ports.
  * **-N** Do not execute a remote command.
  * **-T** Disable pseudo-terminal allocation
  * **-o** Is a way to pass options to the ssh connection overriding the local ssh_config file
  * **-o VerifyHostKeyDNS=no** Specifies whether to verify the remote key using DNS and SSHFP resource records
  * **-o StrictHostKeyChecking=no** This flag is set to **no** to prevent the ssh client refusing connecting to the remote host in case there is an issue with the keys stored in the local ~/.ssh/known_hosts file
  * **-o &#8220;ServerAliveInterval 10&#8221;** Timeout interval before closing the connection if there is no response
  * **-o &#8220;ExitOnForwardFailure yes&#8221;** Exit with error if you cannot set-up the tunnel, even if the ssh connection is working properly
  * **-R 8090:localhost:8080 remoteuser@remoteserver** Forward to port 8090 of the remote host localhost:8080 connecting with user **remoteuser** to **remoteserver** on port 22
  * **-i** **/full/path/to/your/private/key** Use the private key at this path

**  
Restart=always**

There are multiple values for this config parameter, we are just going to use **always** as we want the reverse tunnel up all the time and if it is not up please retry till it is.

**  
RestartSec=5s**

If the service drops wait 5 seconds before restarting it

## [Install] Section {.wp-block-heading}

**WantedBy=default.target**

This line means that this **Service** is assigned to the **Target** default, the one which is required by the system startup, it would be the equivalent in **systemV** of the rc.local in some respect.



**3 &#8211;** Save the file and use systemdctl to enable and start the new service  
Don&#8217;t forget to tail the system logs to verify the connection is working properly; you will notice that &#8216;.service&#8217; has disappeared from the end of the file, if everything is set up correctly it does not matter, systemd will understand what you are asking it to do.

<pre class="wp-block-code"><code># systemctl enable /etc/systemd/system/reversetunnel.servername

# systemctl start /etc/systemd/system/reversetunnel.servername</code></pre>



## Debug and tests {.wp-block-heading}

If all is gone right you should have:

**On your local system** your service running on port 8080 ( use &#8216;**netstat -ntpl**&#8216; to verify ) and an ssh connection to the remote host ( use &#8216;**netstat -nat**&#8216; and &#8216;**ps xxaawwuuu | grep ssh | grep remoteuser**&#8216; to verify )  


**On the remote server** you should see a localhost service listening on 8090 ( use &#8216;**netstat -ntpl**&#8216; to verify ) and if you try to &#8216;**telnet localhost 8090**&#8216; you should be able to see a response from your local system

 [1]: http://thedjbway.b0llix.net/daemontools.html