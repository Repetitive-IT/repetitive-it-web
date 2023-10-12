---
title: "Connect to an SSH Server Without Password"
type: post
date: 2023-10-10T18:20:14+02:00
draft: false
---
Or otherwise said 

How to set up a public-private authentication between your client and a remote server and log-in without any password.

<p class="has-text-align-center">
  &#8212;
</p>

<p class="has-drop-cap">
  Being able to log-in into a server without password is the first step to understand, create and make work your automations.
</p>

It does not really matter if you are executing them with Python or Bash or using Ansible; it is what it is, the fundamental step to understand how systems can securely communicate together.

Password are amazing things, and at the same time terrible things if you get them wrong, asymmetric keys instead are a bit better in terms of usage in a system-to-system scenario, painful at the same level if you get them wrong or loose them.

What to take in consideration before you start

When you create an asymmetric pair of keys, you will finish up having a private and a public key, both should be by default in your home directory under .ssh/

If you want you can provide a different path, but be careful, while the public key is made to be copied/pasted around servers and systems inside the authorized_keys file, the private key has to remain protected and stored securely, installing it only in systems you are going to use and protected with permission 600 to avoid other users malicious eyes to read it.

Let&#8217;s move into the how to do it:

  * From your home directory give the command &#8216;ssh-keygen&#8217; like showed below.
  * Do not change the default path, just press Enter
  * Do not enter a passphrase, just press Enter twice

<pre class="wp-block-code"><code>me@mycomputer:~$ ssh-keygen 
Generating public/private rsa key pair.
Enter file in which to save the key (/home/me/.ssh/id_rsa): 
Created directory '/home/me/.ssh'.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/me/.ssh/id_rsa
Your public key has been saved in /home/me/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:ZOW6KfiX4kGQ/Fu0Lb2hUoRZO/wEtYTa5DmlKvG2y9I me@mycomputer
The key's randomart image is:
+---&#91;RSA 3072]----+
|        oo+      |
|   . . =o*..     |
|    + o=O++      |
|    .o.=*O       |
|     oo.S.=      |
|    .o++ = o     |
|    .+=.+..      |
|    .oE+o        |
|     o=+         |
+----&#91;SHA256]-----+

me@mycomputer:~$</code></pre>

If you see something like the console above then the process is completed and you will be able to find your keys in ~/.ssh/

**id_rsa** being the **Private Key**  
**id_rsa.pub** being the **Public key**

Now, make sure you can connect to the remote server using the ssh client and your password, and let&#8217;s make sure you accepted the host as trusted:

<pre id="block-acd6f396-0a2c-46dd-ad50-7debaf55b2c2" class="wp-block-code"><code>me@mycomputer:~$ ssh remoteuser@remoteserver
The authenticity of host 'remoteuser@remoteserver (192.168.0.80)' can't be established.
ECDSA key fingerprint is SHA256:3/Xkni9+Ecs1eEFyc0uUrnRGB2Vg9nSF/XO2y8teppw.
Are you sure you want to continue connecting (yes/no/&#91;fingerprint])? &lt;TYPE yes&gt;
remoteuser@remoteserver's password: &lt;TYPE YOUR PASSWORD&gt;
remoteuser@remoteserver:~$</code></pre>

Ok, after this we know we are able to log-in and to work on the configuration of the remote server.  
Let&#8217;s do it then, let&#8217;s bring the Pubic key in the remote server&#8230;

There is a fancy way of doing it, but we will do it the not that fancy way just to understand how this system works and we are going to do everything from the same console on the local client in order to avoid confusion.

First let&#8217;s check if the authorized_keys file exists, this file is storing Public Keys for all the users authorized to authenticate and we do not want to delete it, so: 

<pre id="block-fa567133-1718-47b5-898a-5c40744fba38" class="wp-block-code"><code>me@mycomputer:~$ ssh remoteuser@remoteserver ls -all .ssh/authorized_keys
remoteuser@remoteserver's password: &lt;TYPE YOUR PASSWORD&gt;
-rw-r--r-- 1 remoteuser remoteuser 569 Aug  5 16:41 .ssh/authorized_keys
me@mycomputer:~$ </code></pre>

If it does exists like in this case, it may contain someone&#8217;s else key, and because of that we will need to be careful; this means to APPEND our public key at the bottom of the file, rather than overwriting the entire file.  
If the result from the previous command is nothing then we will have to create our file with the correct permissions, please follow from **[HERE][1]**

<p id="WE-ARE-READY-NOW">
  Let&#8217;s copy our Public Key into the home directory of the remote server with the scp command
</p>

<pre id="block-e2f88cf3-c0ce-45a5-9e31-5e9fd57c3e02" class="wp-block-code"><code>me@mycomputer:~$ scp .ssh/id_rsa.pub remoteuser@remoteserver:.
remoteuser@remoteserver's password: &lt;TYPE YOUR PASSWORD&gt;
id_rsa.pub                                                                            100%  566   765.5KB/s   00:00
me@mycomputer:~$</code></pre>

Once the key is there we can, from our local computer, push the content of the file we just copied into the authorized_keys file, but let&#8217;s back-up the file first and verify it has executed correctly

<pre id="block-db89522b-8504-4779-99e5-e3792d7823a8" class="wp-block-code"><code>me@mycomputer:~$ ssh remoteuser@remoteserver 'cp .ssh/authorized_keys .ssh/bkp_authorized_keys'
remoteuser@remoteserver's password: &lt;TYPE YOUR PASSWORD&gt;
me@mycomputer:~$ ssh remoteuser@remoteserver 'ls -all .ssh/ ; diff .ssh/authorized_keys .ssh/bkp_authorized_keys'
remoteuser@remoteserver's password: &lt;TYPE YOUR PASSWORD&gt;
total 16
drwxr-x--- 2 remoteuser remoteuser 4096 Dec  2 11:18 .
drwxr-xr-x 4 remoteuser remoteuser 4096 Dec  2 11:14 ..
-rw-r--r-- 1 remoteuser remoteuser  569 Aug  5 16:41 authorized_keys
-rw-r--r-- 1 remoteuser remoteuser  569 Dec  2 11:18 bkp_authorized_keys
me@mycomputer:~$ 
</code></pre>

Cool, this means we have a backup, let&#8217;s go ahead and inject the Public Key into the authorized_keys file:

<pre id="block-e042278d-fa37-4a84-a6ac-bcd1edf991fb" class="wp-block-code"><code>me@mycomputer:~$ ssh remoteuser@remoteserver 'cat id_rsa.pub &gt;&gt; .ssh/authorized_keys'
remoteuser@remoteserver's password: &lt;TYPE YOUR PASSWORD&gt;
me@mycomputer:~$</code></pre>

It looks like everything worked out as expected, now we should be able to log-in into our remote server without password request.

Let&#8217;s try a command we already issued:

<pre id="block-db89522b-8504-4779-99e5-e3792d7823a8" class="wp-block-code"><code>me@mycomputer:~$ ssh remoteuser@remoteserver ls -all .ssh/authorized_keys
-rw-r--r-- 1 remoteuser remoteuser 569 Aug  5 16:41 .ssh/authorized_keys
me@mycomputer:~$</code></pre>

All done, we do have now a local system able to authenticate without password, but with asymmetric keys.

Way more handy in terms of automation!

<div class="wp-block-group is-layout-flow wp-block-group-is-layout-flow">
  <div class="wp-block-group__inner-container">
    <h3 id="DOES-NOT-EXIST">
      What to do if the authorized_keys file does not exist?
    </h3>
    
    <p>
      If the file authorized_keys does not exist in your system there are a couple more step before we go ahead.<br />When configuring asymmetric authentication it is really important the way we set the permissions on each file and 600 is the number you want to remember.
    </p>
    
    <p>
      Let&#8217;s create the remote file and assign the correct permission then:
    </p>
  </div>
</div>

<pre id="block-db89522b-8504-4779-99e5-e3792d7823a8" class="wp-block-code"><code>me@mycomputer:~$ ssh remoteuser@remoteserver 'touch .ssh/authorized_keys && chmod 600 .ssh/authorized_keys'
remoteuser@remoteserver's password: &lt;TYPE YOUR PASSWORD&gt;
me@mycomputer:~$</code></pre>

This should do the trick, a file with permission 600 (read and write only for the owner of the file) to store the keys for the remoteuser authentication.

Let&#8217;s verify all is gone as we expect:

<pre id="block-e2f88cf3-c0ce-45a5-9e31-5e9fd57c3e02" class="wp-block-code"><code>me@mycomputer:~$ ssh remoteuser@remoteserver ls -all .ssh/authorized_keys
remoteuser@remoteserver's password: &lt;TYPE YOUR PASSWORD&gt;
-rw-r--r-- 1 remoteuser remoteuser 569 Aug  5 16:41 .ssh/authorized_keys
me@mycomputer:~$ </code></pre>

Perfect, we are now ready to go back and inject our Public Key into the newly created file,let&#8217;s take it from **[HERE][2]**

 [1]: #DOES-NOT-EXIST
 [2]: #WE-ARE-READY-NOW