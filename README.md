# advent-of-cyber-2022
## Day 1

The first challenge wants to get you familiar with different frameworks.

Going to the site attached to the challenge we are presented with a puzzle, you can complete it just using basic table puzzle solving skills but the correct answer is the steps in the first stage of the Unified Kill Chain framework:

    Reconnaissance -> Weaponisation -> Delivery -> Social Engineering -> Exploitation -> Persistence -> Defence Evasion -> Command and Control

It then takes you to another puzzle and the answer is stage 2 of the Unified Kill Chain framework:
  
    Pivoting -> Discovery -> Privilege Escalation -> Execution -> Credential Access -> Lateral Movement

Finally it takes you to a third puzzle where the answer is stage 3 of the Unified Kill Chain framework:
    
    Access -> Collection -> Exfiltration -> Impact -> Objectives
 
 This gives us the answer for both questions:
  
  *Who is the adversary that attacked Santa's network this year?* The Bandit Yeti<br>
  *What's the flag that they left behind?* THM{IT'S A Y3T1 CHR1$TMA$}

## Day 2

The second challenge wants to get you familiar with log parsing.

The first step is to ssh into the box provided.

The first challenge is to find how many log files there are in the home directory: `2`                            
The second challenge is to identify the webserver logfile: `webserver.log`
The Third challenge is to identify the day santa's naught and nice list were stolen:

At first I cat'd the file to see what format I was working with. It looks like a standard webserver log file.
    
    cat webserver.log
    
While skimming the file I noticed that there are alot of 404 errors so I used grep to filter only logs with a 200 response code

    cat webserver.log | grep 200
    
This leads to filtering both logs with 200 as a responce code and logs with 200 in the GET request so I made my grep a bit more specific

    cat webserver.log | grep \"\ 200
    
Which leads to very few logs to actually show up and I was able to immediatly identify the log I needed but for posterity I filtered for the specific log

    cat webserver.log | grep \"\ 200 | grep santa
   
Also I noticed that most of the logs have "gobuster", which is a directory bruteforcer, as their user agent and was curious how many logs were actually real which lead me to do a reverse grep
    
    cat webserver.log | grep -v gobuster
    
It turns out there are only 3 real logs so I over grepped here.<br>
For the answer I had to get a hint since I didn't pick up it wanted the day and *not* the date even though it said day the colloquial use of 'day' is 'date'. Anyway the answer is `Friday`

The next two challenges aren't anything fancy it just asks for the attacker's ip and the name of the file which are: `10.10.249.191` and `santaslist.txt` respectivly.

The final part is to find a flag in the standard THM{} format. Chekhov's gun tells us it's going to be in the log file we haven't taken a look at yet

    cat SSHD.log | grep THM
    
And sure enough it is the onlything to come back: `THM{STOLENSANTASLIST}`

## Day 3

This challenge is all about OSINT

The first step is to identify the Registrar for "santagift.shop":

    whois santagift.shop
    
This command will give you the registrar but you have to massage the output to the format TryHackMe wants: `Namecheap Inc`

The next step is to find the site's source code and find some creds in a file:
The essay before the challenges talkes about google dorking but you don't actually have to dork to find this repo 

    searching "github santagift.shop" will bring up https://github.com/muhammadthm/SantaGiftShop
    
Since we are looking for creds pulling up config.php is a good place to start. <br>
In this file we can see usernames, passwords, and the flag `{THM_OSINT_WORKS}`

The next step askes what the file name is: `config.php`

The next step wants to know the name of the QA server: Searching the config file we find it is `qa.santagift.shop`

The final step asks what password is used for the QA and PROD server: Searching the config file we can find that the password is reused `S@nta2022`

## Day 4

This challenge is all about scanning. It want's you to use nmap but I think nmap would be overkill for these challenges.

The first step is to find the name of the webserver running on the provided box:

    nc <ip> 80
    
Using this command we can grab the banner of the webserver which tells us it's an `Apache` server.

The next question askes what service runs on port 22. This is usually `ssh`

The next question askes you to find the flag in the smb share. The first step is to enumerate the smb share: 

    smbclient -L \\\\<IP>
    
This shows 2 non default shares: "sambashare" and "admins"

    smbclient \\\\10.10.47.241\\admins -U ubuntu%S@nta2022
    
Checking the admins share using the credentials we found in yesterday's challenge and we get 2 files: "flag.txt" and "userlist.txt"

    more flag.txt
    
Using this command in the smb shell will give us the contents of the file: `{THM_SANTA_SMB_SERVER}`

The next question askes what the password is for the user "santahr":
    
    more userlist.txt
    
With this command we can dump the contents of the file and find the password for "santahr" is: `santa25`

## Day 5

This one is all about bruteforcing.

The first question want's us to find the vnc password. Since we can see how many characters the password is we can process the rockyou wordlist to only have passwords that have 8 characters.

    grep -E "^.{8}$" rockyou.txt | sort > rockyou-8char.txt 
    
This oneliner uses regex to select the beginning of the line eight of any character and the end of the line. It then sorts it and puts it in a new file

We can then use that new wordlist in hydra to automate the bruteforcing
    
    hydra -P rockyou-8char.txt vnc://<IP> -V -f
    
This gives us the password `1q2w3e4r`

The last question wants us to grab the flag on the desktop of the remote box. Theres nothing fancy here you just login using the password and the wallpaper has `THM{I_SEE_YOUR_SCREEN}`

## Day 6

This challenge is about email parsing. I'm not a fan of this challenge because it forces you to use their vdi instead of just giving you the file to analyze. Their vdi doesn't passthrough ctrl sequences which makes it difficult to use the terminal so I ended up exfiltrating the file to my own machine using netcat.

The first question asks what who the sender of the email is.

    cat Urgent\:.eml | grep -i from
    
This gives us `chief.elf@santaclaus.htm`

The next question asks what the return address is.

    cat Urgent\:.eml | grep -i return-path
   
This gives us `murphy.evident@bandityeti.thm`

The next question asks on who's behalf the email was sent. This was answered with the first command: `Chief Elf`

The next question asks what the x-spam score is.

    cat Urgent\:.eml | grep -i spam
    
Which gives us `3`

The next question asks what the message-id is.

    cat Urgent\:.eml | grep -i message-id

Which gives us: `QW9DMjAyMl9FbWFpbF9BbmFseXNpcw==` which converts to `AoC2022_Email_Analysis`

The next question wants to know, using the service provided, what the sender's email reputation is. Thankfully this site is curl-able:

    curl emailrep.io/chief.elf@santaclaus.htm

While this would give us all the info we would need in the real world the challenge wants a single word that is only found on the website through a browser with a js engine, something curl doesn't have. The answer is `Risky`

The next question wants to know what the name of the attachement is.
    
    cat Urgent\:.eml | grep -i x-attached
    
Which gives us `Division_of_labour-Load_share_plan.doc`

The next question wants to know what the hash of the attachment is. It doesn't specify what algorithm to use but in the story text it talks about sha256 so thats what I'm going to use.

    cat Urgent\:.eml | grep -i content-disposition -A 1058 | head -n -1 | tail -n +3 | sed -e 's/^M//g' | base64 -d > out.doc && sha256sum out.doc

This is a long one-liner that is doing quite a bit. I used `wc -l Urgent\:.eml` before this one liner to see how many lines were in the file. I then cat the file, grep for "content-disposition" and extend that to the max line count, this gives the attachment base64 encoded plus some encapsulation text. It then gets piped to head which in this context removes the last line then gets piped to tail which in this context removes the first 2 lines. I then remove all breaklines using sed. It should be noted `^M` is not literal, so you can't directly copy/paste this command, its my terminal's ascii representation of a breakline. You can get this by pressing ctrl+v and then enter in your terminal to get this. It then gets decoded and since .doc is a binary file, as opposed to .docx which is an xml based textfile although nowadays it's zipped by default, the output needs to be redirected to a file since outputting a binary file can messup your terminal. I then use && instead of & since the former will wait for the previous command to be done. This gives us `0827bb9a2e7c0628b82256759f0f888ca1abd6a2d903acdb8e44aca6a1a03467`

The next question wants us to input the hash into virus total, navigate to the behaviour section and find the second tactic marked in the Mitre ATT&CK section. Nothing fancy here just copy/paste and the answer is `Defense Evasion`

The final question wants us to navigate to InQuest, input the hash, and find the subcatagory for the file. `macro_hunter` 


## Day 7

This challenge is about using cyberchef. Unfortunatly this is **only** about using cyberchef all the questions about analyzing the maldoc are centered around cyberchef so I can't using any commandline magic.

The first question is what is the version of cyberchef on the vm. `9.49.0`

The next question asks how many recipies it requires to extract urls from the maldoc. This question is one of the reasons I don't like challenges that contrain you to doing something a certian way. I was able to find the url's with 8 recipies, technically 7 since the last one only highlights what I need, but the answer is `10`. My recipie was this:

    Strings -> Find/Replace \[\_\] -> Drop bytes 124 -> From Base64 -> Decode Text UTF-16LE -> Find / Replace ['()+"`] -> Find / Replace b2H_s https -> Regex Url

The next question asks what the file name for the malware that gets downloaded is. In the first url we can see it ends with `mysterygift.exe`

The next question askes what the last defanged url for the bandityet domain is. After adding the defang recipie we can see it is `hxxps[://]cdn[.]bandityeti[.]THM/files/index/`

The last question asks what the golden ticket domain is. The question actually lies to you about the format. It states the format is "Domain/<GOLDEN_FLAG>" so in my mind it should be "secretSanta.THM/Goldenticket/THM_MYSTERY_FLAG" this has both the domain and the subdirectories but the actual answer is just the subdirectory `THM_MYSTERY_FLAG` without a domain.
