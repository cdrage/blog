---
layout: post
category: smartphone
title: "Switching from a voice plan to data only in Canada"
date: 2017-01-30 16:31
---

I recently switched to a $20 / month 1GB data-only plan for my phone. Gone are the days of $70+ bills. Data only. No built-in SMS / phoning. Only awesome data.

Ironically, if you ask anywhere if they have a data-only plan for cell-phones, they would reply no.. However, no one is going to stop you from subscribing to a tablet data-only plan and sticking the SIM card in your phone.

Canadians still pay the highest (in the WORLD!) for cellphones [according to CBC](http://www.cbc.ca/news/business/cellphone-deals-canada-1.3587744). Seriously. The cheapest plan that you can find is $45 / month for 500MiB of data on [Koodo](https://koodomobile.com). I researched on how I could switch and found [this blog post](http://http://misener.org/ditched-voice-plan-went-data/) and decided why-not. The next day, I cancelled my Koodo plan and switch to a Bell data-only tablet plan.

Nothing's every 100%. I switched with some caveats introduced with a data-only plan:

  - I phone via a SIP provider [(VOIP.ms)](http://voip.ms) on a SIP client [(Linphone)](https://play.google.com/store/apps/details?id=org.linphone&hl=en)
  - I SMS via [VOIP.ms SMS](https://play.google.com/store/apps/details?id=net.kourlas.voipms_sms&hl=en) an [open source](https://github.com/michaelkourlas/voipms-sms-client) Android app that connects to VOIP.ms' free SMS service
  - My SIM-card assigned phone number works, but has extremely-high rates (50c a minute, 75c per text). Ideally you'd only use this for emergencies if you're without data!

I won't go into detail on how to set this up, but here's the gist of what you need to do:
  
  1. Get rid of that pesky old plan
  2. Walk into Bell and ask for a tablet data-only flex plan
  3. Throw that old SIM card out and put in your new one
  4. Setup a DID on [VOIP.ms](http://voip.ms) via [Step 1](https://www.loganmarchione.com/2014/06/part-1-setting-voip-number-voip-ms/) and [Step 2](https://www.loganmarchione.com/2014/08/part-2-setting-voip-number-voip-ms/) of this [guide](https://www.loganmarchione.com/2014/06/part-1-setting-voip-number-voip-ms/). Afterwards, use a client such as [Linphone](https://play.google.com/store/apps/details?id=org.linphone&hl=en) or [CSipSimple](https://play.google.com/store/apps/details?id=com.csipsimple&hl=en) with your DID registration details. This will be your new way to phone people as well as your new number
  5. Enable SMS on VOIP.ms with [this guide on their wiki](https://wiki.voip.ms/article/SMS)
  6. Download [VOIP.ms SMS](https://play.google.com/store/apps/details?id=net.kourlas.voipms_sms&hl=en) and enter your login details. It's text-only (no MMS or group chats!) but it's the best way to communicate to those still using non-smartphone plans. Besides, this will be your communication medium to convince them to switch to Hangouts / WhatsApp / Messenger / etc..

It only took an hour for setup and a bit of troubleshooting for initially setting up the DID on VOIP.ms (is there ANY service out there with free SMS texting that's as clean as VOIP.ms w/ VOIP.ms SMS?)

But now I can relish in the fact that my phone bill is $20 a month (tax-in!) and no longer $65+. Thanks to the flex plan that Bell has, if I decide to go over the 1GB, it's only $40 / month for 5GB of data. Any GB after that is $10 / month. Reminds me of [Google Fi](https://fi.google.com) (plz come to canada!).

