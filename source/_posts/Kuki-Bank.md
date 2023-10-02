---
title: Kuki-Bank
date: 2023-10-02 16:28:18
author: w0rmhol3
categories: Write-Up
tags: Web CTF
---
SKR CTF is a good platform to practice CTF challenges and test out cybersecurity knowledge. The challenge done is a medium level web challenge called Kuki Bank.
`titleimg1`
From the challenge it can be seend that the goal is to exploit a simple banking website that is under beta testing, with every created account is given RM100 for free. This challenge had given a hint for the challenger to ease its understanding.
`hintimg`
The hint tells that one of the key findings can be within the transfer feature, so now let's go and have a look at the system.
`loginimg`
Once entered the website, the system introduces its login page. To start, we need to create an account. 
`homepageimg`
Once completed the registration and had proceeded with the login, the homepage will show the balance of the account. To get the flag, a total of RM13,333,337 will be deducted from the account, hence we need to find a way to increase the account balance. 
`transferimg`
Going to the transfer page, we can see that it will be able to transfer the balance from the logon account to another account. All it requires is the account number and the amount to be transferred.
`test2homepageimg`
Knowing that, a second account must be created, hence account Test2 is used to figure out how the transaction feature works within the system. This account will be used to transfer the balance to the first account. 
`test2transfersuccessimg`
When inputted the account number of the first account and an amount less than the balance available, it will show that the transferred is successful.
`test2balance1img`
Hence, just to check if the balance is actually deducted, I went back to the homepage of the second account, and it shows the balance have only RM0.01 left. This makes me wonder, what if I do another transfer, will the transfer be accepted?
`test2transfersuccessimgagain`
Hence, I tried again and WALLA, the transfer is still accepted. 
`test2balance2img`
The balance will be in negative within the Test2 account. To exploit this system, we need a method to automate the amount transfer from Test2 account to the primary account. There are different methods that can achieve this in which can be creating a script that can non-stop creating transfer from account A to B, or use a tool. In this case, I used burpsuite to automate the process directly.
`intruder1img`
All we need to do is intercept the traffic in which the transfer is made, and send it to intruder. As we just need to automate the transfer feature, we do not need to modify the payload. Hence, the payload can be inputted anywhere that does not affect the packet.
`intruder2img`
Going to the payload tab, we need to change the payload type to “Null payload” as we do not need to modify anything within the packet. Last thing to set is to change the Payload settings to “Continue indefinitely” to allow the transfer to keep on going without stop. Once all the settings are done, we can click on start attack.
`intruder3img`
As you can see, the process has been automated and will send the packets non-stop.
`testbalanceimg`
When you visit the primary account again it will show that the balance had been increased a lot.
`test2balanceimg`
As for the Test2 account that keeps on transferring its debit to the primary account, the amount will keep going deeper in the negative value.
`intruder4img`
The exploit does not stop here, Once you get the negative value of the transferring account to -1000 balance, you can keep on transferring 999, at -10,000 you can  transfer 9,999, and getting to -100,000, you can keep on transferring 99,999. So on burpsuite, all we need to do is to change the transfer amount within intruder and start the attack again with the same configuration mentioned previously.
`transferlimitimg`
Keep in mind, the limit per transfer is only at 100,000 or else even using burpsuite, the amount will not be increased. So we can modify the transfer amount to as much as the limit only.
`flagimg`
Once it reached the required amount of to get the flag, click on the `Redeem Flag` button and you will get the flag.
`FLAG: SKR{Bu5in35S_L0gic_3rRoR_889460}` 
