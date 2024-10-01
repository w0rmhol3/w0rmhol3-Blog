---
title: 'SKR CTF Web Challenge: Kuki-Bank'
date: 2023-10-02 16:28:18
author: w0rmhol3
categories: Write-Up
tags: Web CTF
cover: https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/SKRCTF/Web/Kuki%20Bank/bank.jpg
---
SKR CTF is a good platform to practice CTF challenges and test out cybersecurity knowledge. The challenge done is a medium level web challenge called Kuki Bank. <!--more-->

![Kuki-Bank](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/SKRCTF/Web/Kuki%20Bank/Kuki%20Bank.png)

From the challenge it can be seen that the goal is to exploit a simple banking website that is under beta testing, with every created account is given RM100 for free. This challenge had given a hint for the challenger to ease its understanding.

![Hint](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/SKRCTF/Web/Kuki%20Bank/Hint.png)


The hint tells that one of the key findings can be within the transfer feature, so now let's go and have a look at the system.

![Login Page](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/SKRCTF/Web/Kuki%20Bank/login1.png)


Once entered the website, the system introduces its login page. To start, we need to create an account. 

![Homepage](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/SKRCTF/Web/Kuki%20Bank/test%20account.png)


Once completed the registration and had proceeded with the login, the homepage will show the balance of the account. To get the flag, a total of RM13,333,337 will be deducted from the account, hence we need to find a way to increase the account balance. 

![Transfer](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/SKRCTF/Web/Kuki%20Bank/test%20balance1.png)


Going to the transfer page, we can see that it will be able to transfer the balance from the logon account to another account. All it requires is the account number and the amount to be transferred.

![Test2 Account](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/SKRCTF/Web/Kuki%20Bank/test2%20account.png)


Knowing that, a second account must be created, hence account Test2 is used to figure out how the transaction feature works within the system. This account will be used to transfer the balance to the first account. 

![Transfer Successful](https://github.com/w0rmhol3/w0rmhol3.github.io/assets/91303166/b3a06e32-5459-4a67-abbf-5cd2d71bdd3d)


When inputted the account number of the first account and an amount less than the balance available, it will show that the transferred is successful.

![Account Balance](https://github.com/w0rmhol3/w0rmhol3.github.io/assets/91303166/ae3ecc82-1a40-4dc4-9306-c1a63fd25e4e)


Hence, just to check if the balance is actually deducted, I went back to the homepage of the second account, and it shows the balance have only RM0.01 left. This makes me wonder, what if I do another transfer, will the transfer be accepted?

![Transfer Successful](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/SKRCTF/Web/Kuki%20Bank/test2%20transfer%20success.png)

Hence, I tried again and WALLA, the transfer is still accepted. 

![Balance](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/SKRCTF/Web/Kuki%20Bank/negative%20balance.png)


The balance will be in `negative` within the Test2 account. To exploit this system, we need a method to automate the amount transfer from Test2 account to the primary account. There are different methods that can achieve this in which can be creating a script that can non-stop creating transfer from account A to B, or use a tool. In this case, I used `burpsuite to automate the process` directly.

![Burpsuite Intruder](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/SKRCTF/Web/Kuki%20Bank/intruder1.png)


All we need to do is intercept the traffic in which the transfer is made, and send it to `intruder`. As we just need to automate the transfer feature, we do not need to modify the payload. Hence, the payload can be inputted anywhere that does not affect the packet.

![Intruder](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/SKRCTF/Web/Kuki%20Bank/intruder2.png)


Going to the payload tab, we need to change the payload type to `“Null payload”` as we do not need to modify anything within the packet. Last thing to set is to change the Payload settings to `“Continue indefinitely”` to allow the transfer to keep on going without stop. Once all the settings are done, we can click on start attack.

![Intruder3](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/SKRCTF/Web/Kuki%20Bank/intruder3.png)


As you can see, the process has been automated and will send the packets non-stop.

![Primary Balance](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/SKRCTF/Web/Kuki%20Bank/test%20balance1.png)


When you visit the primary account again it will show that the balance had been increased a lot.

![Test2 Balance](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/SKRCTF/Web/Kuki%20Bank/test2%20balance2.png)


As for the Test2 account that keeps on transferring its debit to the primary account, the amount will keep going deeper in the negative value.

![Intruder4](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/SKRCTF/Web/Kuki%20Bank/intruder4.png)


The exploit does not stop here, once you get the negative value of the transferring account to `-1000 balance`, you can keep on transferring `999`, at `-10,000` you can  transfer `9,999`, and getting to `-100,000`, you can keep on transferring `99,999`. So on burpsuite, all we need to do is to `change the transfer amount` within intruder and start the attack again with the same configuration mentioned previously.

![Transfer Limit](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/SKRCTF/Web/Kuki%20Bank/transfer%20limit.png)


Keep in mind, the `limit per transfer is only at 100,000` or else even using burpsuite, the amount will not be increased. So we can modify the transfer amount to as much as the limit only.

![Flag](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/SKRCTF/Web/Kuki%20Bank/flag.png)


Once it reached the required amount of to get the flag, click on the `Redeem Flag` button and you will get the flag.
`FLAG: SKR{Bu5in35S_L0gic_3rRoR_889460}` 
