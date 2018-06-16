CTF Rules
=========
The CCIT CTF Finals '18 is a security competition for students who attended the CyberChallenge.IT training program. The competition follows the typical attack/defence format. An introduction to this game setting can be found on the [FAUST CTF](https://2018.faustctf.net/information/attackdefense-for-beginners/) website (although no VPN is required to play this CTF).

Access to the competition is granted to the 8 teams of universities involved in the CyberChallenge.IT project. Every team is composed by 4 selected participants of each local CCIT CTF that took place on Jun 7. Teams are required to declare a captain among them who will be in charge of handling the communication between her/his team and the organisers.

The rules listed on this page may change as more issues are raised by the participants. Also, the organizers keep the right to change them at any time. Keep in mind that it is not possible/feasible to list all the rules and the exceptions to rules that apply to the CTF competition. When in doubt, use common sense or ask. Lastly, the aim of the CTF is not to determine the best team in terms of technological supremacy. The goal is to share experience and knowledge in the computer security and also to have some fun together.


Schedule
--------
The contest will take place on Jun 27 from 9:00am to 7:30pm (Italian local time) with the actual CTF competition expected to last for 8 hours. A detailed schedule is provided below:

* 9:00am - 10:00am: access to the VMs is granted, but the network is closed. Teams should use this time to analyse services before everyone can attack each other
* 10:00am - 5:00pm: the netowork is open! Flags are dispatched to each service by the gameserver and teams can earn points by submitting proofs of successful exploitation (flags)
* 5:15pm - 6:30pm: 10-minute talks by each team
* 6:30pm - 7:30pm: jury evaluation


Talks
-----
Following the experience of the European Cyber Security Challenge 2017 (ECSC17), the CyberChallenge.IT committee decided to recognize the importance of soft-skills by evaluating short talks as part of the score achieved by each team. Teams are required to provide a short presentation of 10 minutes focusing on one of the services exploited during the CTF. Presentations will be provided in Italian and evaluated by a jury in terms of clarity, correctness and - most importantly - ability of presenting the problem to a non-technical audience.


Network and Setup
-----------------
The game is played within the `10.10.0.0/16` subnet. Each team has its own vulnerable machine located at `10.10.<team_id>.1`, while players connecting to the game network are assigned an ip in the range `10.10.<team_id>.150` - `10.10.<team_id>.200`. VMs can also be reached via the DNS name `team<team_id>`. A graphical representation of the network scheme will be added soon to the rules.

The _manager_ machine is responsible for dispatching flags to the vulnerable machines, checking services integrity, hosting the scoreboard and updating scores. Participants are asked to attack vulnerable machines of other teams to retrieve proofs of successful exploitation (flags). Flags must be submitted to the flag submission service hosted by the organisers to score points. At the same time, teams must defend the vulnerable services installed on their VMs. Teams can do whatever they want within their network segment.

Internet access is granted to install new software on the VM and on the laptops of participants, if needed. Due to environmental constraints (remember that the competition will take place inside a museum!) the bandwidth will be limited: try to get your laptop ready with most tools already installed and avoid wasting time during the game to download large amount of data. For the same reason, orgnanisers discourage interaction between CTF network and remote servers (e.g., starting attacks from Google cloud): large computational resources are not required to succeed at the competition.

Beware that if you mess up your vulnerable machine, all we can do is reset it to its original state (backup your exploits, tools and patches!).

There could be some ssh backup keys deployed by organisers on all vulnerable boxes. Feel free to remove them, but that will make life harder for you in case of hotfixes to be released. Legitimate public keys will be announced as part of the rules.


Scoring
-------
The game is divided in _rounds_ (also called _ticks_) having the duration of 120 seconds. During each round, a bot will add new flags to your vulnerable machine. Moreover it will check the integrity of services by interacting with them and by retrieving the flags through a legitimate accesses. 

Your team gains points by attacking other teams, defending its own vulnerable machine and by keeping services up and running. The total score is the sum of the individual scores for each service. The score per service is made up of three components:

* Offense: Points for flags captured from other teams and submitted to the gameserver within their validity period
* Defense: Points for not letting other teams capture your flags
* SLA: Points for the availability and correct behavior of your services

For each service, the component scores for a team are calculated as in this Python-like pseudocode:

### Offense

```Python
offense = count(flags_captured_by[team])
for flag in flags_captured_by[team]:
    offense += (1 / count(all_captures_of[flag]))
```

### Defense

```Python
defense = 0
for flag in flags_owned_by[team]:
    defense -= sqrt(count(all_captures_of[flag]))
```

### SLA

```Python
sla = (count(ticks_with_status['up'] + 0.5 * ticks_with_status['recovering'])) * sqrt(count(teams))
```

### Total Score

```Python
total = 0
for service in services:
    total += offense[service] + defense[service] + sla[service]
```


Flags
-----
A flag is a string made up of 31 uppercase alfanumeric chars, followed by `=`. Each flag is matched by the regular expression `[A-Z0-9]{31}=`. 

To manually submit a flag, click on the _flag submission service_ option in the top-right menu of the CTF portal after logging-in and enter the flag in the input form. During the CTF, anyway, you may want to automatically submit flags. To do so, you can submit stolen flags by performing an HTTP POST request to the gameserver at `10.20.0.1`. The request must contain the keys `flag` and `team_token`, where the value of the first entry is the stolen flag and `team_token`is a string that allows the server to identify your team. The token of your team can be found in your _team page_ on the CTF portal. **Important: do not perform user authentication whilst submitting flags, this is not needed and may overload our servers**.

As an example, we provide a simple python snippet that accounts for the submission of an hardcoded flag.

```Python
#!/usr/bin/python
import requests

url = 'https://10.20.0.1/submit'
team_token = '<your_token>'
stolen_flag = 'QWERTYUIOPASDFGHJKLZXCVBNM01234='
 
r = requests.post(url, data={'team_token': team_token, 'flag': stolen_flag}
```


Technical and Human Behaviour
-----------------------------
We'd like everyone to enjoy a fair game. For this reason we ask you to follow these simple rules:

* No attacks against the infrastructure (this website and challenges) including denial-of-service (DOS), floods, DNS poisoning, ARP spoofing, MITM, etc...</li>
* The only permitted attack targets are the vulnerable machines! Players are not allowed to attack each other (e.g., you can't break into rivals' laptops)
* Destructive behavior and unfair practices are severely prohibited. These include removing flags or deleting files on compromised hosts, creating fake flags to break legitimate attacks
* Network vulnerability scanners - with the exception of `nmap` - are not allowed, do something better
* Sharing flags, exploits or hints between teams is severely prohibited and will grant you the exclusion from the competition.

Before attempting to break one of the aforementioned rules, remember that all the network traffic is logged.


Communication
-------------
We invite everyone attending the contest to join us on slack. The invitation link to the organization will be provided in your profile page! Challenge hints, if any, will be announced there.


Credits
-------
This year the CTF is organized by the [c00kies@venice](https://secgroup.github.io) hacking team under the direction of the [CyberChallenge.IT](https://cyberchallenge.it) committee.

Organisers would like to thank [RuCTFe](https://ructfe.org/rules/) and [FAUST CTF](https://2018.faustctf.net) for inspiring part of these rules.


And remember to...
------------------
...have fun and hack all the things!!!!!11