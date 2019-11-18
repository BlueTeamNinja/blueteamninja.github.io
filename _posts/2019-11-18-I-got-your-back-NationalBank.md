---
title: I got your back, NationalBank
date: 2019-11-18 18:30
categories:
  - blog
tags:
  - security
  - cyber
  - news
  - industry
  - pitchforks
published: true
---
## Letter to the Journalist

> This is a copy/paste of the email I sent to the reporting journalist on November 18, 2019

Good Afternoon, 

I just came across your article ( [https://www.cbc.ca/news/canada/nova-scotia/national-bank-canada-customer-banking-privacy-1.5334059](https://www.cbc.ca/news/canada/nova-scotia/national-bank-canada-customer-banking-privacy-1.5334059) ) along with some scathing twitter comments regarding National Banks practices.  If NationalBank was indeed asking for the login credentials on their platform then I would 100% agree with the sentiments in the article.  However, in this case, I believe it is inciting a misunderstood and completely unfair response to an organization that is actually doing a good thing.  Also, the comments of both experts indicate a lack of understanding - either of the platforms being used, or what is actually happening during the creation of an account on NationalBanks website.

They are *NOT* asking you to give them credentials to another institution.  While they have provided a user experience that may seem like that, what they are doing is utilizing SecureKey (A Canadian company) a 'Credential Broker' to do an authentication to one of their partners, known as a 'Credential Provider'.  What this means is that my information ONLY exists with the financial institution of my choosing.  This is the same concept as 'Login with Facebook / Google+ / Twitter / Microsoft Account' on many many sites.  SecureKey proxies the request and provides a platform for institutions to receive a fairly trusted 'Yes / No' that you have a verified account at a reputable instituion.  To be analogous, this is the fundamental equivalent of calling my neighbour to ask if my car is in the driveway.  Neither you, nor my neighbour have access to my car - but you can conclude that my car is here. 

Could National Banks PR have cleared this up?  Probably - but that's not my area of expertise.  What is my expertise, is that I both design secure networks and break into secure networks (as a professional service).  What they (NationalBank) are doing to verify identity is right - or at least as close to right as we can get in the Cybersecurity industry.

What this means, is that they are using an intermediary system that will send a login request out to any of its partners - when that request is approved (your details and password were correct) or rejected (they weren't), then SecureKey simply returns a yes or no back to the originator - in this case NationalBank.  Furthermore, when using this system - you are clearly redirected to your own institutions page.  If I choose to use CIBC to verify my identity while opening a NationalBank account, I am logging into what is clearly a CIBC page (again, where I have trusted my information to reside), then returned to NationalBank to continue the process.  This is a heavily tested and monitored system.  No lock is unpickable, but using the 'Economy of Scale' alone, having nearly every financial institution in Canada involved with serious financial ramifications of failure - it is likely a better than any individual institute can do. 

Furthermore, you quoted the former privacy commissioner stating, "National Bank's request for customer login information for another bank is 'unprecedented.' - I can now conclude that she doesn't file her own taxes - as it uses the same mechanism - and has for several years.  As do many federal services, and many financial institutions.

Step 1:  Go to CRA My Account
![](/assets/images/2019-11-18-13-57-04.png)

Step 2:  Verify your identity by logging into your financial instution
![](/assets/images/2019-11-18-13-57-28.png)


Also, to finish off.  I believe the IPC could take notes from how NationalBank conducts their identity verification.  As a basic threat and risk mitigation technique - the less places that my information exists, the better.  Data breaches in 2019 are a numbers game.  It ultimately happens to nearly every organization - so it is far better off to play the odds.  I would rather have my bank protect my details AND act as a credential provider. 

The online form used to file an IPC complaint is asking me for details that could easily be confirmed by a credential provider such as SecureKey.  This would be both safer, more secure, and more convenient as a user of the service.  Going to the url: [https://www.ipc.on.ca/guidance-documents/forms/file-a-privacy-complaint-under-fippa-mfippa/](https://www.ipc.on.ca/guidance-documents/forms/file-a-privacy-complaint-under-fippa-mfippa/) - You are asked to enter several personal details.  While this page is covered with information on my information is used and stored by other parties, none of it covers how the IPC handles and safeguards my data.  They make the rules on the topic, but don't provide technical guidance on how to carry it out - in other words, Judges aren't cops and vice versa. What is the IPC doing with my information - other than providing it to those who ask nicely?  I'd rather use SecureKey. 

The Privacy and Access Council of Ontario is in a similar position.  While they didn't request any information from me to create an account, they didn't even confirm that I own the email address I entered.   I could have signed up as you and I was sent straight to the 'My Account' page where I could subscribe to whatever I chose; again, I could have done this with your email address.  Ontario also has an in-house developed (assuming) technology called One-KEY.  It is the same concept but without anywhere near the level of technical investment behind it.  [https://www.one-key.gov.on.ca/iaalogin/browserreq.jsp](https://www.one-key.gov.on.ca/iaalogin/browserreq.jsp)  If you look at the browser requirements, they are suggesting 13-year old web browsers, and the two guidelines on blindly and globally enabling Java and Cookies are very bad advice.  They were sound guidelines at the time, but the page hasn't been modified since 2011 according to the footer.  I would recommend they follow NationalBanks practice by viewing the information on the website alone.  Sharon Polsky was quoted as saying, "... less invasive and risky, such as credit bureau checks".  This requires the collector at AT LEAST collect my address, date of birth, usually SIN - in other words, most of the information I consider private - then the collecting institution (the same one with security advice pre-dating the original iPhone) is going to store and process this information.  Now there are 2 organizations with my information stored.  Lastly, according to [https://www.canada.ca/en/financial-consumer-agency/services/credit-reports-score/credit-report-score-basics.html](https://www.canada.ca/en/financial-consumer-agency/services/credit-reports-score/credit-report-score-basics.html) - this would result in an 'inquiry' being recorded on my history.  This is literally the opposite of 'less invasive'.  Opening multiple bank accounts shouldn't negatively affect my credit rating, as multiple inquiries does, according to the very next sentence in the referenced link.

In summary, the privacy experts slamming National Bank don't seem to be fully aware of how crucial these key technologies are to protecting Canadians in 2019.  Stirring up the pitchforks among the masses only makes it harder to do what us network security practitioners do, and it makes it a LOT harder to train and explain to users what is and isn't a good idea.  Most of these institutions are government or financial bodies that should be highly trusted by all users but I think we have work to do before that is a reality.

In my opinion, what NationalBank is doing is perfectly fine and both of the represented institutions could learn some lessons from them.  

Thanks for your time. 

-Abe