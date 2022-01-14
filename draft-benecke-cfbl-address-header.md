---
title: "Complaint Feedback Loop Address Header"
abbrev: "CFBL Address Header"
docname: draft-benecke-cfbl-address-header-05
category: exp

ipr: trust200902
area: art
workgroup: Network Working Group
keyword: Internet-Draft

stand_alone: yes
smart_quotes: no
pi: [toc, sortrefs, symrefs]

author:
  ins: J. Benecke
  name: Jan-Philipp Benecke
  organization: CleverReach GmbH & Co. KG
  street: Schafjueckenweg 2
  city: Rastede
  code: 26180
  country: Germany
  phone: +49 4402 97390-16
  email: jpb@cleverreach.com

normative:
  XARF:
    title: eXtended Abuse Reporting Format
    author:
      org: Abusix
    date: false
    seriesinfo:
      Web: https://github.com/abusix/xarf

informative:

--- abstract

This document describes a method that allows an email sender to specify a complaint feedback loop (FBL) address as an email header.
Also it defines the rules for processing and forwarding such a complaint.
The motivation for this arises out of the absence of a standardized and automated way to provide mailbox providers with an address for a complaint feedback loop.
Currently, providing and maintaining such an address is a manual and time-consuming process for email senders and providers.

It is unclear, at the time of publication, whether the function provided by this document has widespread demand, and whether the
mechanism offered will be adopted and found to be useful. Therefore, this is being published as an Experiment, looking for a constituency
of implementers and deployers, and for feedback on the operational utility. The document is produced through the Independent RFC stream
and was not subject to the IETF's approval process.

--- middle

# Introduction and Motivation
For a long time there has been a way for a mailbox provider to forward manual complaints back to the email sender.
The mailbox provider provides what is called a feedback loop {{?RFC6449}}. 
This feedback loop is used to give operators of, e.g. broadcast marketing lists, feedback on resulting complaints from their marketing mailings.
These complaints are based on manual user interaction, e.g. IMAP movement to "junk" or by clicking on a "This is spam" button.

As described in {{?RFC6449}} the registration for such a feedback loop needs to be done manually by a human at any mailbox provider who provides a FBL.
This can be quite time-consuming if there are new feedback loops rising up, or the email sender wants to add new IP addresses or DKIM domains.
In addition, a manual process is not well suited and/or feasible for smaller mailbox providers.

Because of the manual process involved, the email sender has to go through all providers again, delete his existing subscriptions and register with their new complaint address.

This document addresses this issue with a new email header.
It extends the recommendations for the complaint feedback loop described in {{?RFC6449}} with an automated way to submit the necessary information to mailbox providers.

Mail senders can add this header, willing mailbox providers can use it to forward the generated report to the specified complaint address.
The email sender only needs to add an email header and does not need to manually register with each feedback loop provider.
The elimination of a manual registration and verification process would be another advantage for the mailbox providers.

A new email header has been chosen in favour of a new DNS record to easily distinguish between
multiple broadcast marketing list operators / email senders without requiring user or administrator intervention.
For example, if a company uses multiple mailing systems, each system can set this header itself without requiring any change by the users or administrators within their DNS.
No additional DNS query is required on the mailbox provider side to obtain the complaint address.

This document has been prepared in compliance with the GDPR and other data protection laws to address the resulting issues
when providing an automated address for a complaint feedback loop, as the email may contain personal data.

Nevertheless, the described mechanism bellow potentially permits a kind of man-in-the-middle attack between the domain owner and the recipient.
A bad actor can generate forged reports to be "from" a domain name the bad actor is attacking and send this reports to the complaint FBL address.
These fake messages can result in a number of actions, such as blocking of accounts or deactivating recipient addresses.
This potential harm and others are described with potential countermeasures in [](#security-considerations).

In summary, this document has the following objectives:

* Allow email senders to signal that a complaint address exists without requiring manual registration with all providers.
* Allow mailbox providers to obtain a complaint address without developing their own manual registration process.
* Be able to provide a complaint address to smaller mailbox providers who do not have a feedback loop in place
* Provide a GDPR compliant option for a complaint feedback loop.

## Scope of this Experiment
The CFBL-Address header and the CFBL-Feedback-ID header are an experiment. 
Participation in this experiment consists of adding the CFBL-Address-Header on email sender side or by using the CFBl-Address-Header to send FBL reports to the provided address on mailbox provider side.
Feedback on the results of this experiment can be emailed to the author or raised as an issue at https://github.com/jpbede/rfc-cfbl-address-header/.

The goal of this experiment is to answer the following questions based on real-world deployments:

- Is there interest among email sender and mailbox providers?
- If the mailbox provider adds this capability, will it be used by the senders?
- If the email sender adds this capability, will it be used by the mailbox provider?
- Does the presence of the CFBL-Address/CFBL-Feedback-ID field introduce additional security issues?
- What additional security measures/checks need to be performed at the mailbox provider before a complaint report is sent?
- What additional security measures/checks need to be performed at the email sender after a complaint report is received?

This experiment is considered successful if the CFBL-Address header has been implemented by multiple independent parties (mail sender and mailbox provider)
and these parties successfully use the address specified in the header to exchange feedback loop reports.

If this experiment is successful and these headers prove to be valuable and popular, it may be taken to the IETF for
further discussion and revision. One possible outcome could be that a working group creates a specification for the standard track.

## Difference to One-Click-Unsubscribe
For good reasons, the One-Click-Unsubscribe {{?RFC8058}} signaling already exists, which may have several interests in common with this document.
However, this header requires the List-Unsubscribe header, whose purpose is to provide the link to unsubscribe from a list.
For this reason, this header is only used by operators of broadcast marketing lists or mailing lists, not in normal email traffic.

The main interest of this document now is to provide an automated way to signal mailbox providers an address for a complaint feedback loop.
It is the obligation of the email sender to decide for themselves what action to take after receiving a notification; this is not the subject of this document.

# Definitions
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this 
document are to be interpreted as described in BCP 14 {{!RFC2119}} {{!RFC8174}} when, and only when, they appear in all capitals, as shown here.

The keyword "CFBL" in this document is the abbreviation for "complaint feedback loop" and will hereafter be used.

The keyword "MBP" in this document is the abbreviation for "mailbox provider", it is the party who receives an email, and will be used hereafter.

The keyword "mail sender" in this document is used to describe the party who sends an email, this can be an MBP, a broadcast marketing list operator or any other email sending party. It will be used hereafter.

# Requirements

## Received email {#received-email}
This section describes the requirements that a received email, i.e. the email that is sent from the email sender to the MBP and about which a report is to be sent later, must meet.

### Simple
If the domain in the From: header {{!MAIL=RFC5322}} and the domain in the CFBL-Address header are identical, this domain MUST be covered by a valid {{!DKIM=RFC6376}} signature.
This signature MUST meet the requirements described in [](#received-email-dkim-signature).

The following example meets this case:

~~~
Return-Path: <sender@mailer.example.com>
From: Awesome Newsletter <newsletter@example.com>
To: me@example.net
Subject: Super awesome deals for you
CFBL-Address: fbl@example.com; report=arf
Message-ID: <a37e51bf-3050-2aab-1234-543a0828d14a@mailer.example.com>
Content-Type: text/plain; charset=utf-8
DKIM-Signature: v=1; a=rsa-sha256; d=example.com;
       h=Content-Type:Subject:From:To:Message-ID:
       CFBL-Feedback-ID:CFBL-Address;

This is a super awesome newsletter.
~~~

### Complex
If the domain in From: header {{!MAIL=RFC5322}}, of the email sent by email sender to MBP, differs from the domain in the CFBL-Address header, 
the domain of the CFBL-Address header MUST be covered by an additional valid {{!DKIM=RFC6376}} signature.
Both signatures MUST meet the requirements described in [](#received-email-dkim-signature).

This double DKIM signature ensures that both the domain owner of the From: domain and the domain owner of the CFBL-Address: domain
agree to receive the complaint reports on the address from the CFBL-Address: header.

The following example meets this case:

~~~
Return-Path: <sender@super-saas-mailer.com>
From: Awesome Newsletter <newsletter@example.com>
To: me@example.net
Subject: Super awesome deals for you
CFBL-Address: fbl@super-saas-mailer.com; report=arf
Message-ID: <a37e51bf-3050-2aab-1234-543a0828d14a@mailer.example.com>
Content-Type: text/plain; charset=utf-8
DKIM-Signature: v=1; a=rsa-sha256; d=example.com;
       h=Content-Type:Subject:From:To:Message-ID:
       CFBL-Feedback-ID:CFBL-Address;
DKIM-Signature: v=1; a=rsa-sha256; d=super-saas-mailer.com;
       h=Content-Type:Subject:From:To:Message-ID:
       CFBL-Feedback-ID:CFBL-Address;

This is a super awesome newsletter.
~~~

### DKIM signature {#received-email-dkim-signature}
The CFBL-Address header MUST be included in the "h=" tag of the aforementioned valid DKIM-Signature.
When the CFBL-Feedback-ID header is used, it MUST also be included in the "h=" tag of the aforementioned valid DKIM signature.

If the domain has neither the required coverage by a valid DKIM signature nor the required header coverage by the "h=" tag, the MBP SHALL NOT send a report email.

## Report email
The report email (sent by MBP to email sender) MUST have a valid {{!DKIM=RFC6376}} signature. 
The aforementioned valid DKIM signature MUST cover the From: header {{!MAIL=RFC5322}} domain, from which the report is sent to the email sender. 

If the message does not have the required valid {{!DKIM=RFC6376}} signature, the email sender SHALL NOT process this complaint report.

As part of this experiment, it is recommended to determine what plausibility and security checks are useful and achievable.

# Implementation

## Email senders {#mail-senders}
An email sender who wishes to receive complaints about their emails MUST include a CFBL-Address header in their messages.

The receiving complaint FBL address specified in the messages MUST accept {{!ARF=RFC5965}} compatible reports by default.
The email sender can OPTIONALLY request a {{XARF}} compatible report if they want one, as described in [](#xarf-report).
The MBP MAY send a {{XARF}} compatible report if it is technically possible for them to do so, otherwise a {{!ARF=RFC5965}} compatible report will be sent.

It is strongly RECOMMENDED that these reports be processed automatically. Each sender must decide for themselves what action to take after receiving a report.

The email sender MUST take action to address the described requirements in [Requirements](#requirements).

## Mailbox provider {#mailbox-provider}
If the MBP wants to process the complaints and forward it, they MUST query the CFBL-Address header and forward the report to the complaint FBL address.

By default, an {{!ARF=RFC5965}} compatible report MUST be sent when a manual action has been taken e.g., when a receiver marks a mail as spam, 
by clicking the "This is spam"-button in any web portal or by moving a mail to junk folder, this also includes {{?IMAP=RFC9051}} and {{?POP3=RFC1939}} movements.
The MBP SHALL NOT send any report when an automatic decisions has been made e.g., spam filtering. 

The MBP MUST send a {{XARF}} compatible report when the email sender requests it as described in [](#xarf-report).
If it is not possible for the MBP to send a {{XARF}} compatible report as requested, a {{!ARF=RFC5965}} compatible report MUST be sent.

The MBP MUST validate and take action to address the described requirements in [Requirements](#requirements).

# Complaint report {#complaint-report}
The complaint report MAY be a {{XARF}} report if the email sender requests it, and it is technically possible for the MBP to do so, otherwise the complaint report MUST be a {{!ARF=RFC5965}} report.

The report MUST contain at least the Message-ID {{!MAIL=RFC5322}}. If present, the header "CFBL-Feedback-ID" of the complaining email MUST be added additionally.

The MBP MAY omit all further headers and/or body to comply with any data-regulation laws.

It is highly RECOMMENDED that, if used, the CFBL-Feedback-ID includes a hard to forge component such as an {{?HMAC=RFC2104}} using a secret
key, instead of a plain-text string. 

## XARF compatible report {#xarf-report}
A email sender wishing to receive a {{XARF}} compliant report MUST append "report=xarf" to the [CFBL-Address-Header](#cfbl-address-header).
The resulting header would look like the following:

~~~
CFBL-Address: fbl@example.com; report=xarf
~~~

# Header Syntax

## CFBL-Address {#cfbl-address-header}
The following ABNF imports fields, WSP, CRLF and addr-spec from {{!MAIL=RFC5322}}.

~~~ abnf
fields /= cfbl-address

cfbl-address = "CFBL-Address:" 0*1WSP addr-spec
               [";" 0*1WSP report-format] CRLF

report-format = "report=" ("arf" / "xarf")
~~~

## CFBL-Feedback-ID
The following ABNF imports fields, WSP, CRLF and atext from {{!MAIL=RFC5322}}.

~~~ abnf
fields /= cfbl-feedback-id

cfbl-feedback-id = "CFBL-Feedback-ID:" 0*1WSP fid CRLF

fid = 1*(atext / ":")
~~~

# Security Considerations {#security-considerations}
This section discusses possible security issues, and their possible solutions, of a complaint FBL address header.

## Attacks on the FBL address
Like any other email address, a complaint FBL address can be an attack vector for malicious emails.
For example, complaint FBL addresses can be flooded with spam.
This is an existing problem with any existing email address and is not created by this document.

The email sender must take appropriate measures.
One possible countermeasure would be a rate limit for the delivering IP.
However, this should be done with caution; normal FBL email traffic must not be affected.

## Automatic suspension of an account
Sending an FBL report against a mailbox can cause the account holder to be unreachable if an automatic account suspension occurs too quickly.
An example: someone sends an invitation to his friends. For some reason, someone marks this mail as spam.
Now, if there is too fast automatic account suspension, the sender's account will be blocked and the sender will not be able to access his mails.

MBPs and email senders must take appropriate measures to prevent this.
MBPs and email senders therefore have, mostly proprietary, ways to assess the trustworthiness of an account.
For example, MBPs and email senders may take into account the age of the account and/or any previous account suspension before suspending an account.

## Enumeration attacks / provoking unsubscription 
A malicious person may send a series of spoofed ARF messages to known complaint FBL addresses and attempt to guess a Message-ID/CFBL-Feedback-ID or any other identifiers.
The malicious person may attempt to mass unsubscribe/suspend if such an automated system is in place.
This is also an existing problem with the current FBL implementation and/or One-Click Unsubscription {{?RFC8058}}.

The sender of the received email must take appropriate measures.
As a countermeasure, it is recommended that the CFBL-Feedback-ID, if used, use a hard-to-forge component such as a {{?HMAC=RFC2104}} with a secret
key instead of a plaintext string to make an enumeration attack impossible.

If it is impossible for the email sender to use a component that is difficult to fake, they should take steps to avoid enumeration attacks.

## GDPR and other data-regulation laws
The provision of such a header itself does not pose a data protection issue.
The resulting ARF report sent by the MBP to the email sender may violate a data protection law because it may contain personal data.

This document already addresses some parts of this problem and describes a privacy-safe way to send a FBL report.
As described in [](#complaint-report), the MBP can omit the entire body and/or header and send only the required fields.
Nevertheless, each MBP must consider for itself whether this implementation is acceptable and complies with existing privacy laws.

As described in [](#complaint-report), it is also strongly RECOMMENDED that the Message-ID and, if used, the CFBL-Feedback-ID.
contain a component that is difficult to forge, such as a {{?HMAC=RFC2104}} that uses a secret key, rather than a plaintext string.
See [](#hmac-example) for an example.

Using HMAC, or any other hard to forge component, ensures that only the email sender has knowledge about the data.

## Abusing for Validity and Existence Queries
This mechanism could be abused to determine the validity and existence of an email address, which exhibits another potential privacy issue.
Now, if the MBP has an automatic process to generate a complaint report for a received email, it may not be doing the mailbox owner any favors.
As the MBP now generates an automatic complaint report for the received email, the MBP now proves to the email sender that this mailbox exists for sure, because it is based on a manual action of the mailbox owner.

The receiving MBP must take appropriate measures. One possible countermeasure could be, for example, pre-existing reputation data, usually proprietary data.
Using this data, the MBP can assess the trustworthiness of an email sender and decide whether to send a complaint report based on this information.

# IANA Considerations

## CFBL-Address
The IANA is requested to register a new header field, per {{?RFC3864}}, into the "Provisional Message Header Field Names" registry:

~~~ abnf
Header field name: CFBL-Address
  
Applicable protocol: mail

Status: experimental

Author/Change controller: Jan-Philipp Benecke <jpb@cleverreach.com>

Specification document: this document
~~~

## CFBL-Feedback-ID
The IANA is requested to register a new header field, per {{?RFC3864}}, into the "Provisional Message Header Field Names" registry:

~~~ abnf
Header field name: CFBL-Feedback-ID

Applicable protocol: mail

Status: experimental

Author/Change controller: Jan-Philipp Benecke <jpb@cleverreach.com>

Specification document: this document
~~~

# Examples
For simplicity the DKIM header has been shortened, and some tags have been omitted.

## Simple
Email about the report will be generated:

~~~
Return-Path: <sender@mailer.example.com>
From: Awesome Newsletter <newsletter@example.com>
To: me@example.net
Subject: Super awesome deals for you
CFBL-Address: fbl@example.com; report=arf
CFBL-Feedback-ID: 111:222:333:4444
Message-ID: <a37e51bf-3050-2aab-1234-543a0828d14a@mailer.example.com>
Content-Type: text/plain; charset=utf-8
DKIM-Signature: v=1; a=rsa-sha256; d=example.com;
       h=Content-Type:Subject:From:To:Message-ID:
       CFBL-Feedback-ID:CFBL-Address;

This is a super awesome newsletter.
~~~

Resulting ARF report:

~~~
------=_Part_240060962_1083385345.1592993161900
Content-Type: message/feedback-report
Content-Transfer-Encoding: 7bit

Feedback-Type: abuse
User-Agent: FBL/0.1
Version: 0.1
Original-Mail-From: sender@mailer.example.com
Arrival-Date: Tue, 23 Jun 2020 06:31:38 GMT
Reported-Domain: example.com
Source-IP: 192.0.2.1

------=_Part_240060962_1083385345.1592993161900
Content-Type: text/rfc822; charset=UTF-8
Content-Transfer-Encoding: 7bit

Return-Path: <sender@mailer.example.com>
From: Awesome Newsletter <newsletter@example.com>
To: me@example.net
Subject: Super awesome deals for you
CFBL-Address: fbl@example.com; report=arf
CFBL-Feedback-ID: 111:222:333:4444
Message-ID: <a37e51bf-3050-2aab-1234-543a0828d14a@mailer.example.com>
Content-Type: text/plain; charset=utf-8
DKIM-Signature: v=1; a=rsa-sha256; d=example.com;
       h=Content-Type:Subject:From:To:Message-ID:
       CFBL-Feedback-ID:CFBL-Address;

This is a super awesome newsletter.
------=_Part_240060962_1083385345.1592993161900--
~~~

## GDPR safe report
Email about the report will be generated:

~~~
Return-Path: <sender@mailer.example.com>
From: Awesome Newsletter <newsletter@example.com>
To: me@example.net
Subject: Super awesome deals for you
CFBL-Address: fbl@example.com; report=arf
CFBL-Feedback-ID: 111:222:333:4444
Message-ID: <a37e51bf-3050-2aab-1234-543a0828d14a@mailer.example.com>
Content-Type: text/plain; charset=utf-8
DKIM-Signature: v=1; a=rsa-sha256; d=example.com;
       h=Content-Type:Subject:From:To:Message-ID:
       CFBL-Feedback-ID:CFBL-Address;

This is a super awesome newsletter.
~~~

Resulting ARF report contains only the CFBL-Feedback-ID: 

~~~
------=_Part_240060962_1083385345.1592993161900
Content-Type: message/feedback-report
Content-Transfer-Encoding: 7bit

Feedback-Type: abuse
User-Agent: FBL/0.1
Version: 0.1
Original-Mail-From: sender@mailer.example.com
Arrival-Date: Tue, 23 Jun 2020 06:31:38 GMT
Reported-Domain: example.com
Source-IP: 2001:DB8::25

------=_Part_240060962_1083385345.1592993161900
Content-Type: text/rfc822-headers; charset=UTF-8
Content-Transfer-Encoding: 7bit

CFBL-Feedback-ID: 111:222:333:4444
------=_Part_240060962_1083385345.1592993161900--
~~~

## GDPR safe report with HMAC {#hmac-example}
Email about the report will be generated:

~~~
Return-Path: <sender@mailer.example.com>
From: Awesome Newsletter <newsletter@example.com>
To: me@example.net
Subject: Super awesome deals for you
CFBL-Address: fbl@example.com; report=arf
CFBL-Feedback-ID: 3789e1ae1938aa2f0dfdfa48b20d8f8bc6c21ac34fc5023d
       63f9e64a43dfedc0
Message-ID: <a37e51bf-3050-2aab-1234-543a0828d14a@mailer.example.com>
Content-Type: text/plain; charset=utf-8
DKIM-Signature: v=1; a=rsa-sha256; d=example.com;
       h=Content-Type:Subject:From:To:Message-ID:
       CFBL-Feedback-ID:CFBL-Address;

This is a super awesome newsletter.
~~~

Resulting ARF report contains only the CFBL-Feedback-ID:

~~~
------=_Part_240060962_1083385345.1592993161900
Content-Type: message/feedback-report
Content-Transfer-Encoding: 7bit

Feedback-Type: abuse
User-Agent: FBL/0.1
Version: 0.1
Original-Mail-From: sender@mailer.example.com
Arrival-Date: Tue, 23 Jun 2020 06:31:38 GMT
Reported-Domain: example.com
Source-IP: 2001:DB8::25

------=_Part_240060962_1083385345.1592993161900
Content-Type: text/rfc822-headers; charset=UTF-8
Content-Transfer-Encoding: 7bit

CFBL-Feedback-ID: 3789e1ae1938aa2f0dfdfa48b20d8f8bc6c21ac34fc5023d
       63f9e64a43dfedc0
------=_Part_240060962_1083385345.1592993161900--
~~~

# Acknowledgments
Technical and editorial reviews and comments were provided by the colleagues at CleverReach, the colleagues at Certified Senders Alliance and eco.de,
Arne Allisat and Tobias Herkula (1&1 Mail & Media) and Sven Krohlas (BFK Edv-consulting).

--- back
