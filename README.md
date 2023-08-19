# fraudzero
The purposes of this project are: 
1. *Fool me once, shame on you. Fool me twice, shame on me.*

   We have to stop relying on _customers_ to find all fraud, that's ridiculous and *isn't happening*, I'm sure my mom doesn't read her bank statement that closely, *nor should she at 86*. The zero fraud is acknowleged to be aspirational, but *if you don't try, you can never succeed* either. We may not be able to catch every fraudulent transaction, but we can make it much, much harder for it to continue. *Perfection is the enemy of the good and the great.*
3. At this point, the industry has to start pooling resources the same way the fraudsters do in the dark web. So we need a central place to meet and store fraud detection recipes and data. Small banks should have the same defenses as large banks, because the small banks aren't the enemy, the fraud is the enemy. 
4. Every bank and bank-like object (PayPal Venmo, etc.) shouldn't have to re-discover the hard way which transactions are fraud. There should be a common pool from which to draw. We should hate the *fraudsters* more than we hate our *competitors*. 

   ## Inspiration
   This project was inspired by 2! netflix charges on my bank statement. One was for $19.99, which was valid. The other was for $21.37, which was just wrong. Netflix only bills once/month and they only bill as follows.

   * With Ads: $6.99/month
   * Standard: $15.49 + $7.99 for 1 optional extra member slots. So the choices are $15.49 and $23.48
   * Premium: $19.99 + $7.99 for up to 2 "extra member slots. So that's $19.99, $27.98 and $35.97
  
     So Chase, my bank, could have found all fraudulent Netflix transactions with the following SQL:
     ``` SQL
     select * from transactions where lower(source) contains "etflix" and value not in [6.99, 15.49, 23.48, 19.99, 27.98, 35.97]
     ```
     They could find out the scope of the problem by replacing ```select *``` with ```select count(*)``` as follows.
     ``` SQL
     select count(*) from transactions where lower(source) contains "etflix" and value not in [6.99, 15.49, 23.48, 19.99, 27.98, 35.97]
     ```

     A vendor that only bills once/month per account and only 1 of 6 values? This is the definition of low hanging fruit. *Fraudulent Netflix charges shouldn't exist.* The fact that they do is just laziness on the part of Chase and Netflix, for shame. Both Chase and Netflix are large companies, they should work together because the reputation of both of their companies suffer, plus eventually Elizabeth Warren will come after them, and ham-handed interference by Congress is going to be much, much worse than implementing a few database queries. At the very least, they should run the count(*) query above so we have a sense of the scale of the problem. If you work at Chase, do this now, because someday a lawyer will read this and file a class action suit, and your own databases will happily provide them with the information. 
     It will be a cheap investment to participate in this project, and I expect it to reap benefits in excess of $1M/year, **per bank**. Probably more to be honest, the $1M is to include small banks that have even fewer anti-fraud resources than a huge bank like Chase. 1 man/month/year to use the data in this project is relatively cheap. 1 man/year/year for a large bank is even cheaper. 
  
     An additional inspiration for me is that a *bank-like-object* aka *Paypal* have $12,000 in possibly fraudulent transactions in my account since 2019, by someone called *Apple Services*. I orignally thought this was someone generating fraud against my Paypal account, and pretending to be Apple, but come to find out today no such luck, Paypal can't just wipe all the Apple Services transactions. Apple is the *source* of the fraudulent transactions. Someone is pretending to be whoever and generating app store billing that got funneled to PayPal. So the Apple Services vendor is a real account but Apple was very sloppy with their Paypal integration and don't provide a path back to the actual transaction on their side so its an excruciating process to track every AppStore purchase back to a paypal transaction. 
  
     Most of these banks and BLOs set limits in how much fraud they'll eat, which is annoying, but makes too much business sense to me to get too upset about them, *unless they're low hanging fruit*.

     My suggestion, if you happen to know Elizabeth Warren, is that she give them a year from the first shipment/1.0 of this project, and after that point, banks and bank-like-objects are liable for all fraud transactions past that date for any fraud detected by this project. They can't claim they didn't know, we built the tools for them to find them. Pyschic powers not required, just do your friggin job. 

     They'll scream, but many banks don't offer the same protections credit cards have on debit cards, because they *don't have to* and bankers are cheap, but rarely can see the big picture. It was when the government made the banks liable for fraud instead of the consumer that they got serious about security, we're just extending the principle. (YMMV with your bank, I happen to know Chase explicitly provides the same protections on debit cards, though they whine like little bitches over any transaction over 60 days, which cheeses me off, because *Netflix fraud shouldn't exist.*)   

     ## Outputs/Architecture

     * **Query Builder**: A set of queries in a Python query DSL that will generate SQL queries for the database and schema of the local database or data warehouse to detect fraudulent transactions for all of the Fortune and S&P 500. We're using a DSL so we can write the queries in code and then have a downsteam customer pick their database or data warehouse; turn the crank and *Bob's your uncle, you're good to go, just copy these into your BI system or UI scaffolding project*. We're bounding the problem for now to those sources, because as I stated earlier, *perfection is the enemy of the good and the great*. If we prevent fraudsters from spoofing themselves as any of these 500 companies, we'll catch a large portion of the fraud.
     The SP500 gives us the 500 top publically traded companies in the world, while the F500 gives us some additional privately held companies. 
       I'm leaning toward the Q DSL in Django, though there's a challenge there because someone would have to define a Django relational model first, which is more fricton than just running the query builder and putting the queries into your BI tool. But I can probably do that with a yaml definition file and a mako template, so you fill in your column names for your schema in the yaml file, and everything else just works.  
     * **Black/White Lists**: Known fraud destinations so fraudsters have to be more creative about where they squirrel away their ill gotten gains. When I get these blackmail emails *You looked at Porn, and then you touched yourself*, they always have a unique, never used account bitcoin wallet. But in the real world, setting up a bank account requires physical presence so if we go after the fraudsters accounts, we'll gradually drive them even farther underground. When I worked at one company doing some cybersecurity and ecommerce, all the fraud was from one particular country, so we just stopped doing commerce with that country, there weren't enough real transactions to merit the hassle. We did that by editing the country popup so we wouldn't ship there... 
     Not sure what the white lists are for yet, just know that whenever you have a black list, you need a white list too. Things like the "AD10H" fraud can go in here as a blacklist. Oh, wait I do know what the white list is for, that's where we can keep a mapping of routing number and known accounts for each of the F500 and SP500. If you get a transaction and it purports to be "IBM" but its not the appropriate account, that's a no go. That may or may not need encryption so that only the banks and bank-like objects can read the routing numbers and account numbers, but that's just some PKI. The blacklisted accounts can be in plaintext, I really don't care if the fraudsters start raiding each other's accounts. *and the horse they rode in on* is the correct phrase here. 
       
     * **Tools**: These will be reports to run the queries in some sort of web UI or command line tools that spit out HTML to produce the reports on the scope of the problem. Eventually, I want to build tools to implement a force-of-will fraud detection and management workflow platform where someone can review the most common types of known fraud, pick a set to work on, and proceed. So you can hire humans to review the fraud, since humans are really good at that kind of thing. 
       
     * **AI**: I'm kind of an AI skeptic, most uses of AI sound good in the boardroom but in practice amount to *we're going to dust everything with magic fairy dust and find all the fraud*. Hey, I support the idea of milking naive CEOs for a bunch of high salarys for some AI guys, but I'm not going to support some kind of black-box system for detecting fraud.

       I do know that rule systems are fragile, so eventually I want to support some bayesian tests for detecting fraud. I'm ok with those because they're really statistical, not some kind of black box AI system that purports to detect **cancer in X-rays** but actually detects the presence of the **oncologists signature**. True story, not me personally, but its a cautionary AI tale against black box algorithms, when they trained the nueral net to draw a bounding box around the "cancer" they discovered their training set was finding the signature. Oops. Plus the bayesian tests will work nicely with the workflow system, so we can rank potential fraud by $ value time likelihood, because all the banks will underfund the fraud review staff.
       
     * **Tests and Test Data**: I'm committed to everything being able to build and run queries against a sqllite database of test data, possibly scripts to generate that test data. It's just good professional practice.
     
     * **Discussions, Issues, etc.** I'll leverage all the great features GitHub already has to the maximum, plus maybe Discord, Slack and Reddit.
    
       ### Funding
       Basically, there's me, my passion/Don Quixoteness, my anger at the fraud, and my extreme spare time during my recovery from brain surgery. I might setup a GoFundMe or something else when the project is further along and maybe something where you can report fraud in addition to your bank and we can distill it into the project for some kind of nominal fee like $10 to pay the reviewers because *software engineers don't fix bugs, they fix bug reports*, but that's a long way off and a difficult problem. See [Context](#Context)
    
       Or hey, maybe one of these corporations will see my work and chip in. We'll see, all I have is a readme so far. 
    
       #### <a name="Context"></a> Context
       So... on 5/31/2023 I fell and hit my forehead on the soap dish in the bathtub, knocking myself unconscious. My wife called 911. Firemen came, couldn't rouse me either, bustled me off to the local hospital *Sacred Heart*.
       
       *Sacred Heart* does a CT scan because I hit my head hard enough to bleed and scar, discovers a brain tumor the size of a tennis ball at 6cm x 6.4cm x 7cm. *There's your problem* I picture them saying. They toss me on a helicopter, where I finally wake up, annoyed that **someone is bothering me when I'm trying to sleep**. The nice LifeFlight people explain I'm going to Portland, take about 20 minutes. I fall back to sleep, don't get to enjoy the helicopter ride to OHSU (Oregon Health and Science University Hospital in Portland). I plan on asking if I can do a ride along later, but I can't drive right now.
       
       At OHSU, they do a crani-something or other that night to prep for removing the tennis ball on 6/3. On 6/3/2023 they remove the tennis ball. I now have 2 gold plates, 20 titaniam screws and 2 "bushings" in my head. I haven't tested this, but I bet I set off the scanners at the airport. 
       
       Lest you feel sorry for me, realize that this is all a *bad at the time but in retrospect saved my life* set of circumstances so far. The other way to detect a brain tumor is you die...so while embarrasing, embarrassment is better than dying, so I'm good with it.
    
       By the way, total plug here for a startup called [Ezra](https://ezra.com) that will give you a full body MRI for $1950. Your health insurance won't pay for it, because health insurance would rather pay $20,000 for you to get a colonoscopy every year than pay $1950 to prevent you having a $200,000 bill because *insurance companies are stupid*. But I digress and you knew that...
       
       For various reasons relating to my wifes dementia so she's in Palm Springs while I'm in Portland, even though the hospital is ready for me to go home, I'm trapped in the hospital for 40 days because I'm hella goofy, have hallucinations, and basically need 24 hours supervision. My brain tumor was benign, it was the kind that presses in from the outside, so it was the best kind of brain tumor to have, if you have to have a brain tumor. My brain is bascially unsquishing like one of those sponges that comes all squished down, you add water and it turns into a dinosaur... After about 30 days I'm mostly rational and h
       
       Took about 2 weeks for the hallucinations to stop, though I still get them occasionally. Mostly, things I thought I lost to old age like my sense of smell, sense of touch and libido are coming back. So moral of the story, [*get an Ezra scan*](https://ezra.com), and its much better having *had* a brain tumor than having a brain tumor. When you have a brain tumor, you know something is wrong, but you don't know what to do about it, because *your brain isn't working*.
    
       Now, post brain tumor, positive outlook for the future for me, I feel more in control of my life than I have in years, and I'm alive. My relationship with my wife is better than ever, it had gotten pretty bad when Dementia Gal and Tumor Lad were cavorting in our new house and the dogs had fleas, but I realized in the hospital that all my built up resentment was because *neither of our brains were working*. So we have phone dates twice/week on Facetime; its not so bad, we had a long distance relationship for 6 months when we first met. 

       #### Extreme Spare Time
       Yeah, so doctors say its 12 months before I reach complete recovery. I can only play so much Zelda and Quest 2. So what to do, what to do.
       
       My first thought was removing the mediocrity and uselessness of the USNWR college rankings. No one actually uses them to choose their child's college, and they have numerous flaws the most obvious being that its entirely subjective and "cooked" so the same few colleges come out on top, though the worst sin is the latency: colleges are completely unmotivated to improve anything because it takes 4 years for anyone to notice. They could hire Jesus Christ as a professor, no one would notice for at least 4 years. I looked into how college rankings are done, as an old project of mine from when I worked at [Chegg](https://www.chegg.com) the textbook rental service. I had to bow out of that at the time but for awhile it was a personal passion.
       
       We could/can do better. I wrote up a pitch, reached out to contacts at Chegg, no joy they're worried about other things. That's unfortunate, because my approach for measuring academic excellence requires textbook sales or rental as an input, if we don't do it now, it will be delayed for another academic year if it gets done at all. I suppose I could reach out to Amazon, but I don't have enough contacts there, so its not gonna happen. Plus full disclosure, I spent so much time thinking of Amazon as the *enemy* when I worked at Chegg that I find it hard to motivate myself to reach out blindly. 
       
       Then I discovered the fraud in my account, and it pissed me off. No, it didn't piss me off it *enraged* me. *Bogus Netflix charges shouldn't exist.* Telling me that Chase would only refund 3 months of them further annoyed me. So I set out to do something about it. This is what I'm doing, *be the change you want to see in the world* and all that.


       Pierce Trowbridge Wetter III
       Springfield, OR

       8/19/2023, 8:51pm.
       
       

     
     
