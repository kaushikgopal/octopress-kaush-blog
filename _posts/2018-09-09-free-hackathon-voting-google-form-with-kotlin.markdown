---
layout: post
title: Free hackathon vote tabulation using Google Forms & Kotlin
date: 2018-09-09T00:23:06+03:00
canonical-url: https://tech.instacart.com/free-hackathon-vote-tabulation-using-google-forms-kotlin-3c7b7080ea
---

We recently held our semi-annual hackathon at Instacart - the Carrot Wars 2018!
In putting this hackathon together, I noticed a pretty blaring gap - there wasn't a simple (and free) online service that would quickly tabulate the results for a hackathon event. We looked around and found some nifty options, but most of them were a tad bit too expensive for our liking. They also were not setup for a single event use or required a monthly subscription. There other usage restrictions, too - max vote count, concurrent user count, etc.
You'd think there would be at least some option out there, given how popular hackathons are these days. We did some cursory searching but couldn't find something that would work for us.
My co-organizer , admittedly wiser about such things, made it super clear to me, "No KG, we ARE NOT building our own hackathon voting website 2 days before the event! You have more important things to do!". So I set out to do exactly (½ of) just that.
If you're on a time crunch and just want to use this post for a hackathon you're about to run, jump to the "How to use this form for your hackathon" section below.
For the juicy details, please continue reading!

<!-- more -->

# Solution choice

## Google Forms

I wasn't too thrilled about building an online voting form website (hi 2018!) so instead I chose to just leverage Google Forms. Google Forms is pretty easy to use, can handle a bunch of users slamming your form at the same time, provides a decent enough UI and is ridiculously easy to create and get going.

## Kotlin script

The part I found more interesting about this process was the result tabulation. I could have probably gotten Google Forms to dump the results in a Google Sheet. But my excel/sheets expression fu was not strong enough to conconct the required mathematical expression to get the results I wanted. 

So instead of a seizure inducing mathematical sheet expression, I did what came more naturally to me and wrote a script in Kotlin to tabulate the votes and `println` the results nice and cleanly 😎.

I chose Kotlin cause I'm an Android developer and we love Kotlin here at Instacart. Interestingly the hackathon also coincided with the [release of Kotlin 1.2.50](https://blog.jetbrains.com/kotlin/2018/06/kotlin-1-2-50-is-out/) where the Kotlin scripting support was supposed to have been improved.

There's something very satisfying about running a script in a terminal prompt like you would with bash, but without all the unpleasantness of the bash syntax and all the pleasantness of the Kotlin language.



# Voting process:

* The hackathon had 5 categories (based on our core company values) that folks could participate in
* On presentation day, we sent out a google form to all hackathon attendees with the list of final projects. They would pick the 1st, 2nd and 3rd places for each category
* Google Forms allows a pretty convenient way to export these results as a CSV file
* This [Kotlin script](https://github.com/kaushikgopal/kotlin-scripts/blob/master/carrot-wars-tabulate.kts) ingests the CSV, tabulate the results, does some weighting math and prints the results for each category

# How to use this form for your hackathon

## 1. Building the Google Form

I can't recommend enough this Google Forms add-on called [**formRanger**](https://chrome.google.com/webstore/detail/formranger/faepkjkcpnnghgdhiobglpppbfdnaehc?hl=en). formRanger allows you to populate your form from a simple google sheet. 

This was particularly useful for us because we had folks signing up right at the very last moment, wanting to pitch their hackathon project. So up until the very last presentation, the project list was in flux and so the form needed to constantly be modified (we had 5 categories and about 30 pitches. That's 150 entrees that we would have had to keep consistent - no bueno). With formRanger setup this was a single list in a simple google sheet.

You want your questions to be of type **multiple choice grid**. This way you can link a project (rows) to a position i.e. 1st, 2nd and 3rd place (columns).

There's another important option you want set for each question - "**Limit to one response per column**". This will make sure you don't have more than one 1st, 2nd or 3rd place.

## 2. Installing kotlin script

You can find most of the [instructions for setting up the Kotlin script in the github project](https://github.com/kaushikgopal/kotlin-scripts). tl;dr: 

{% codeblock lang:bash linenos:false %}
# install sdkman (if you haven't already)
curl -s "https://get.sdkman.io" | bash
sdk install maven
sdk install kotlin
sdk install kscript
# alternatively with homebrew
brew install maven
brew install holgerbrandl/tap/kscript
{% endcodeblock %}

## 3. Generate the CSV results file

In your google form, you should see a tab called "Reponses". From there you should be able to "download responses (.csv)".

Google Forms has a pretty decent visualization, which might just work for you, but I wanted to add some more sophistication in the tabulation of the results like weighting and maybe filtering based on categories. Basically, moving this to an actual program opens up a whole bunch of possibilities.

## 4. Running the command
    
From a terminal prompt run the command like so:

{% codeblock lang:bash linenos:false %}
kscript kscript carrot-wars-tabulate.kts ~/Downloads/star-wars-demo-results.csv
{% endcodeblock %}

_The very first time you run the script, it'll take sometime as it pulls the necessary dependencies via maven. Give it some time; subsequent runs should be pretty snappy._

Here's what it should look like:

    $ kscript carrot-wars-tabulate.kts ~/Downloads/star-wars-demo-results.csv 

    ******* PROGRAM START ***************** 
    1 folk(s) voted!
    -----------------------
    Category: "Which is your favorite star wars movie overall?
    ----
    STAR WARS: EPISODE IV A NEW HOPE                   ----> 6    <- [1]
    STAR WARS: EPISODE VII THE FORCE AWAKENS           ----> 3    <- [2]
    ROGUE ONE: A STAR WARS STORY                       ----> 2    <- [3]
    -----------------------
    Category: "Which was your favorite SW movie from the original episodes?
    ----
    STAR WARS: EPISODE IV A NEW HOPE                   ----> 6    <- [1]
    STAR WARS: EPISODE VII THE FORCE AWAKENS           ----> 3    <- [2]
    STAR WARS: EPISODE V THE EMPIRE STRIKES BACK       ----> 2    <- [3]
    -----------------------
    Category: "Which was your favorite standalone SW movie?
    ----
    ROGUE ONE: A STAR WARS STORY                       ----> 6    <- [1]
    SOLO: A STAR WARS STORY                            ----> 3    <- [2]
    ******* PROGRAM END ***************** 

__Haters hate all you want, but those are my movie choices!__.

## 4. Resources

* [Demo Google voting form](https://goo.gl/forms/hVl7afuKAKgPFhxy1) (go ahead and vote for your choice!)
* Sample [CSV responses file](https://raw.githubusercontent.com/kaushikgopal/kotlin-scripts/master/star-wars-demo-results.csv)
* [Google sheet](https://docs.google.com/spreadsheets/d/1v94s9nIrJcO53ohKke9nTELWOv9wByWfsTyn-Snw_cI/edit?usp=sharing) used to populate above form
* [Source repo](https://github.com/kaushikgopal/kotlin-scripts#tabulate-hackathon-votes) for Kotlin script that tabulates and prints out the results

------------------------------

_Originally posted this article on the Instacart tech blog. Reproduced here for posterity._