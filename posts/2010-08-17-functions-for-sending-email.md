
Email notifications are handy for when you need to be alerted to an event that happens on your machine. Sometimes I might write a shell script that looks for a specific string in a log file, and I might want it to send me an email. Now most Unix systems come with a command called mail. What frustrates me about this command is that there's no way to tell it which mail server to use. It always assumes that the localhost is the SMTP server. 

<!--more-->

After looking around in vain for an alternative, I finally settled upon writing my own function that sends an email. For this to work, you'll need to have nc (netcat) installed. Netcat can be downloaded [here](http://netcat.sourceforge.net), although your operating system may already come with it installed. For this example, I'm using the one that comes with OS X Snow Leopard. 

Here's the function to send an email using nc:

```bash
#
# This function takes five parameters: subject, message, 
# from address, to address, and the smtp server.
# Here's an example:
#
# send_email "Hey!" "How are things? \
#    Give me a call sometime." \
#    john@doe.com jane@doe.com \
#    my.mail.server.com
#
 
function send_email {
    subject=$1
    message=$2
    from_addr=$3
    to_addr=$4
    smtp_server=$5
 
    nc $smtp_server 25 << EOF
ehlo localhost
mail from: $from_addr
rcpt to: $to_addr
data
Subject: $subject
From: $from_addr
To: $to_addr
 
$message
.
quit
EOF
}
```

Ok so that covers shell scripts. I also do some python scripting and sometimes I want my python script to fire off an email every now and then. Here's a function for that:

```python
import smtplib
def sendmail(subject, body, fromaddr, toaddrs, smtp_server):
    server = smtplib.SMTP(smtp_server)
    header = ('Subject: %s\r\nFrom: %s\r\nTo: %s\r\n\r\n' %
        (subject, fromaddr, ', '.join(toaddrs.split())))
    email = ''.join([header, body])
    server.sendmail(fromaddr, toaddrs.split(), email)
    server.quit()
```

Same as before, takes five parameters, however this one allows you to specify more than one recipient so long as the email addresses are delimited by spaces. That's it, toss those in your own library for easy re-usability if you like, or extend them further to meet your needs.
        
