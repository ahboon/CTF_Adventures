---
layout: post
title:  "Junior Defender Camp"
date:   2017-12-29 16:54:07 +0800
categories: jekyll update
---


## Junior Defender Camp 2017 Blog

Finally, Junior Defender Camp 2017 has ended. it was a really great learning experience. Secure coding was covered during the work shop on day 1, but the main event was day 2, the "Attack and Defence" CTF. 

A little background on the game format that they adopted. Each team was assigned 2000 points for asset points, and 200 points for service points. There were two type of flags, non-root flags which was 150 points each, and root flags which was 250 points each. A special challenge was also given, which was worth 500 points, but later increased to 1500 points as no one had solved it. For every comprimised non-root or root flag, the team that had their server comprimised would have a deduction towards their assest point based on what type of flag was comprimised, asset score could go negative as well. As for the attacker who comprimised the flag, they would be awarded attack points based on the type of flag that was comprimised. A deduction of 100 points will be issued to teams who failed to meet service checks every 15min. At the end, the total score is calculated based on `(Attack Score + Asset Score + Service Score)`.

There were 11 teams:

|           **Teams**            |
|:--------------------------:|
|           1.  Insects          |
|          2.  Not Found         |
|         3.  NUSGreyhats        |
|          4.  sHrACK           |
|           5.  Nullsec          |
|       6.  Ingress Shield       |
| 7.  tbd'); DROP TABLE Teams;-- |
|          8.  bOt            |
|            9.  Rojak           |
|        10. Access Denied       |
|           11. SYNers           |


From what I can recall, team "Not Found" consists of SIT students, and team "NUSGreyhats" consists of NUS students (duh), and I could not remember remainder teams. I was from team "Insects", and we were all from the same institution. 

The compeition focused heavily on Web vulnerabilities such as Injection and Misconfigured web apps.
Competition started at 9am, but my team only arrived 15mins before the start. So we did not have enough preperation time. As soon as the competition started, we were still reading the compeition booklet, while the other teams have already started their enumerations. Luckily my team had the roles sorted out. Since it was a team of four, two was on defence, one on offence, and one on hybrid (assisting both  offence and defence). Since I was on the offence, this blog would mainly be on my recounts during my offence phase.

Since the primary attack vector was Web, my team had a huge emphasis on Web App hardening, such as input validation and sanitization for possible SQLi attacks. There were two servers to defence. Server A running on PHP, while Server B running on Django (python). Luckily, the pair that was doing defence for my team were proficient in both PHP and Django web apps, thus making the Web App hardening process an easy task. Meanwhile,  I was on the offence. Since I didn't had the time to do multiple enumerations due to the fact that we arrived late, I had `nmap` running on the background. Meanwhile, I started to look for possible SQLi vulnerabilities, as it was still in the early stages of the competition. On Server A, where it was running "Furniture Store" Web App, there was a products listing page. My first instinct was to use `' OR 1='1##` in the input field to test for injectable inputs. 

To my surprise, the field was injectable. So I opened another tab and used an online SQLi cheatsheet for MySQL which is my "go to site" at this [link][sqlcheat]. It was all about enumerating the tables. Knowing that `' OR 1=1 UNION SELECT table_schema,table_name FROM information_schema.tables WHERE table_schema != 'mysql' AND table_schema != 'information_schema';##` would give me all the tables, I first put it on a notepad. Because before doing that, I had to find out how many columns did the vulnerable Web App allowed. To test this, what I did was adding `ORDER BY <number>` at the end of the field after `'OR 1=1`.  So after enumerating the tables, there was a table in particular that was obviously the table that contains the flag. That table name was `fl4g`. To further enumerate this table, I used this following query from the cheatsheet to enumerate the columns, `' OR 1=1 UNION SELECT table_schema, table_name, column_name FROM information_schema.columns WHERE table_schema != 'mysql' AND table_schema != 'information_schema'`. After looking at the output, in the table `fl4g`, there was a column named `flag`. So confirming this, I finally entered this SQL statement for the flag, `' OR 1=1 UNION SELECT 1,2,3,flag,5,6 FROM fl4g;##`. Giving me the flag for that server. Upon further analysis, I realized that Web App is also using URL to pass the SQL requests to whatever backend PHP functions they have. So since Server A are odd numbers, I increase the URL by 2 to injected the next availble server (Since there were 11 of them). This method sped up my attack phase for this particular flag, and I manage to get 9 out of 10 flags (the 11th is ours), as one of the servers were either brough offline, or IPTables were used to block us. 

For the next flag, it was not enumerated by me, but by the defence people. It was discovered that one of our flags is exposed at the web root directory, and it can be accessed via direct URL entry. The directory name was a custom directory name, thus when `nikto` was used, it did not show up during the enumeration process. The directory was called "bonus", and the flag was called `.bonus_flag`. To confirm that it was accessible on the other servers, I did a test on one of the target server that I was working on. When the flag showed up on the test victim, it was just repeating the change of URL to grab flags and submit via the submission portal. Yet another surge in points. 

Additionally, a file called `.docroot_flag` was exposed in the web root. Knowing that this was a common vulnerability, I repeated my attack pattern to gain the flags from the servers managed by the other teams. But I was only able to retrieve less than six of it, as most of the servers are either harderned or brough offline due to the fact that many of the teams wanted to reduce damage to their assest points. 

Server B (Python Based), was inaccessible throughout the first two hours of the CTF as many of them brought it down to prevent people from attacking it. It was only at 11am, where polling servers would be polling for their services, all Server B of participants were up. At the moments leading to 11am, my team's defence duo had already listed out the vulerable parts of Server B. The very first was an exposeed flag at the root of the Web App, where a document called `.flag.txt` was accessible through the browser. Knowing that at 11am all Server B would be up, I prepared chrome tabs with the URL inserted into the text field. Once it was 11am, all I did was a quick refresh on all chrome tabs, giving me all the team's Server B .flag.txt flag.

During the Web App patching phase, the defence duo also identified the locations of the flags in the database (Since were had access to our own Web App during harderning phase, other teams would have the similar locations as well). This helped me to narrow down which specific tables and columns to inject in order to get the flag. For Server B, the Web App was a Django based Web App that was running a "Cupcake Store" Web App. There was no injection points without getting an "Account" in the Web App, so I created an account an proceeded to find functions that had signs of `"SELECT ....."` functions (Like listing of items, or search fields). I ended up with a page that allows the user to search for cupcakes. But hey, I ain't searching for Cupcakes, I am search for flags! So I repeated the steps of table and column enumeration, and to my surprise, none of the web apps were hardened against SQLi, thus allowing me to get all flags. 

Another vulnerability that was discovered on Server B was that the flag in `settings.py` on Server B could also be exposed at the "create new cupcake" page, where if DEBUG mode for Django was on, an exception error would be shown when an "untrusted input" such as a single quote was entered. It was an unintentional flag exposure, thus this wasn't known to the teams. As this required a user sign in, my team spread the workload evenly for this attack, to speed up the number of comprimised servers, as we anticipate that the other teams might bring their Server B offline again before the next polling period. We were right, as soon as we were done with this attack, about 7 out of 11 teams had their Server B brought down to prevent additional attacks. 


Fast forward into the final 2 hours of the CTF, even though we manage to get a reverse shell on most of the team's Server A through the exploitation of a PHP file that does `ping` or `traceroute` which allowed us to do command injection, we were unable to do previledge escalation successfully. After the compeition, it was revealed by the organizer that the priviledge escalation for this CTF was a `vi priviledge escalation`, an example could be found [here][vipv]. To be honest, I actually came across this method while doing some research. But I did not implement it. So it was kind of a bummer that I did not bother using it, and probably it will be something that I will think of in the future.


So here are some takeaway I had from this CTF:

Speed is important apart from skills, it was because that my team were both quick in comprimising the teams and harderning of Web Apps, which allowed us to maintain the score lead. 

Communication is important, knowing who is doing what, and when is it happening was key to our attack coordination. 

Lastly, never give up. We had a rough start, spending the first 30mins figuring out the infrastructure, and yet the others teams have already gain points. But we maintained our composure and followed the plan we had put out prioir to the CTF.


So to end off this blog, here are the screenshots.

![graph]

![scoreboard] 





[sqlcheat]: http://pentestmonkey.net/cheat-sheet/sql-injection/mysql-sql-injection-cheat-sheet
[vipv]: https://computersecuritystudent.com/UNIX/SUDO/lesson1/
[scoreboard]: https://i.imgur.com/IhlZRXk.jpg
[graph]: https://i.imgur.com/ku77rTz.jpg



