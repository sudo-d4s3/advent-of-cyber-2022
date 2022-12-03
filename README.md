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

### The first step is to ssh into the box provided.

### The first challenge is to find how many log files there are in the home directory: `2`                            
### The second challenge is to identify the webserver logfile: `webserver.log`
### The Third challenge is to identify the day santa's naught and nice list were stolen:

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
