
Several months ago I signed up for Offensive Security's [Cracking the Perimeter](http://www.offensive-security.com/information-security-training/cracking-the-perimeter/) (CTP) course. Having successfully completed the course, I wanted to write a short review on it. CTP focuses primarily on Windows exploit development, while touching a little bit on web application hacking. As CTP is marketed as a non-beginner course, students must complete a registration challenge before they will be allowed to take the course. The challenge itself is relatively easy, if you've done any hacking before, or completed [Penetration Testing with Backtrack](http://www.offensive-security.com/information-security-training/penetration-testing-with-backtrack/) (PWB), it should be pretty straightforward.

<!--more-->

30 days is more than enough time to complete the course modules. There are nine modules, and eight of them are short, taking less than an hour to go through. Exercises are provided at the end of each module, and I highly recommend doing them. You'll be given VPN access to the CTP lab, so make good use it to practice what you learn. To give you a better idea of how quickly it takes to go through the material, I did it at a relaxed pace of one module a day and finished with one week to spare.

The certification challenge is 48 hours long, requiring the student to complete several challenges in order to become OSCE (Offensive Security Certified Expert) certified. To prepare for this, I spent a few weeks recreating exploits on [Exploit-DB](http://www.exploit-db.com/), practicing with different fuzzers, and trying to find bugs. The exam itself is not hard, but it is tricky. I solved all but one challenge, and unfortunately, didn't score enough points on the exam to pass. I was slightly disappointed, but it was a great challenge, and within a month, I had booked a retake. The second time around, I completed all the challenges and received a congratulatory email informing me that I had obtained the coveted OSCE certificate.

So what do you need to know to get the most out of the course?

* You should know enough assembly to be able to write simple programs. You will be looking at a lot of assembly, so you might as well be able to understand most of it. [Assembly Language Step-By-Step](http://www.amazon.ca/Assembly-Language-Step-Step-Programming/dp/0470497025) is a great beginner's book.

* Be familiar with either OllyDbg or ImmunityDebugger. I suggest learning keyboard shortcuts. You will be using it a lot.

* You should be familiar with exploiting vanilla stack buffer overflows. If not, head over to Exploit-DB and download vulnerable applications and practice.

* Have some basic knowledge on web application hacking. Much like the last point, if you're lacking in this department, practice.

For everything else, you'll learn it as you go through the course material. With regards to additional material, I suggest reading through [Corelan's](https://www.corelan.be/index.php/2009/07/19/exploit-writing-tutorial-part-1-stack-based-overflows/) first nine tutorials. For practice, Exploit-DB is a great resource, as well as [OverTheWire](http://www.overthewire.org/wargames/), [SmashTheStack](http://smashthestack.org/), and [Pentester Lab](https://www.pentesterlab.com/).

CTP is a great course, with a fun and challenging exam. If you're interested in learning more about exploitating Windows applications, this is a great course to start with.

