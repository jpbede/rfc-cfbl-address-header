---
title: "Complaint Feedback Loop Address Header"
abbrev: "CFBL Address Header"
docname: draft-benecke-cfbl-address-header-01
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

This document describes a method to allow a mail sender to specify a complaint feedback loop address as an email header
and how a mail receiver can use it. This document also defines the rules for processing and forwarding such a complaint.
The motivation for this arises out of the absence of a standardized and automated way to provide a complaint feedback loop address to mailbox providers.
Currently, providing and maintaining such an address to a mailbox provider is a manual and time-consuming process.

--- middle

# Introduction and Motivation
For a long time there is a way to forward manual complaints back to e.g. a broadcast marketing list operator.
The mailbox provider provides a so-called feedback loop {{?RFC6449}}. This feedback loop is being used to give e.g. operators of broadcast marketing lists 
feedback about resulting complaints from their marketing mailings. Those complaints are based on manual user interaction e.g. IMAP movement to "Junk".

As described in {{?RFC6449}} the registration for such a feedback loop needs to be done manually by a human at any FBL provider he wants to receive complaints from.
Which can be quite time-consuming if there are new feedback loops rising up, or the mail sender wants to add new ip addresses or DKIM domains.
Besides, a manual process isn't well suitable and/or doable for smaller mailbox providers.

The change of such a complaint address e.g. due to an infrastructure change is another problem. 
Due to this manual process the mail sender needs to go through all providers again and delete his existing subscriptions and re-signup with the new complaint address.

This document addresses this problem with a new email header.
It extends the described complaint feedback loop recommendations in {{?RFC6449}} with an automated way to provide the complaint feedback loop address to mail receiver.

Mail senders can add this header and willing mailbox provider can use this header to forward the generated report to the provided complaint address.
The mail sender just needs to add a email header and isn't required to signup manually at every feedback loop provider.
Another benefit would be the mailbox provider doesn't need to develop a manual registration process and verification process.

A new email header has been chosen over a new DNS record in favour to be able to easily distinguish between
multiple broadcast marketing list operators / mail senders, without the intervention of its users or administrators.
For example, if a company uses multiple sending systems, each system can set this header on their own, without the need of a change that has to be done by its users or administrators.
On the side of the mailbox provider, there is no need to do an additional DNS query to get the complaint address.

This document has been created with GDPR and other data-regulation laws in mind and to address the resulting problems 
in providing an automated complaint feedback loop address, as the email may contain personal data.

Summarised this document has following goals:

* Allow mail senders to signal that there is a complaint address without a manual registration at all providers.
* Have a way for mailbox provider to get a complaint address without developing an own manual registration process.
* Be able to provide a complaint address to smaller mailbox providers who do not have a feedback loop registration process.
* Provide a GDPR compliant way for a complaint feedback loop

# Definitions

The keywords "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in BCP 14 {{!RFC2119}} {{!RFC8174}}
when, and only when, they appear in all capitals, as shown here.

The keyword "FBL" in this document is the abbreviation for "feedback loop" and will hereafter be used.

The keyword "CFBL" in this document is the abbreviation for "complaint feedback loop" and will hereafter be used.

The keyword "MBP" in this document is the abbreviation for "mailbox provider" and will hereafter be used.

# Requirements

## Complaining about an email
The email (sent by mail sender/broadcast marketing list operator) about which a complaint should be sent MUST have at least a valid {{!DKIM=RFC6376}} signature.
The aforementioned valid DKIM signature MUST cover at least the Complaint-FBL-Address header domain. 
It is RECOMMENDED that the DKIM signature aligns with the Complaint-FBL-Address header domain and the From header {{!MAIL=RFC5322}} domain where possible.
The Complaint-FBL-Address header MUST be included in the "h=" tag of the aforementioned valid DKIM-Signature.

If the message isn't properly aligned, nor it does have the required header coverage by the "h=" tag of a valid DKIM-Signature, the MBP SHALL NOT send a report email.

This ensures that only reports are sent to the complaint address that are based on an authenticated email.

## Report email
The report email (sent by MBP to mail sender) MUST have a valid {{!DKIM=RFC6376}} signature and MUST cover the From header {{!MAIL=RFC5322}} domain.

If the message does not have the required valid {{!DKIM=RFC6376}}, the mail sender SHALL NOT process this email.

It is highly RECOMMENDED that the mail sender does further plausibility checks.

# Implementation

## Mail senders {#mail-senders}
A mail sender that wishes to receive complaints about their mailings MUST place a Complaint-FBL-Address header in the message.
The mail sender MAY place a CFBL-Feedback-ID header in the message out of various reasons.

The receiving complaint FBL address, placed in the message, MUST accept by default {{!ARF=RFC5965}} compatible reports.

The mail sender can OPTIONAL request, as described in [](#xarf-report), a {{XARF}} compatible report.
The MBP MAY send a {{XARF}} compatible report, if it is technical possible, otherwise a {{!ARF=RFC5965}} compatible reports will be sent.

It is highly RECOMMENDED processing these reports automatically. Each mail sender MUST decide on their own which measure they take after receiving a report.

The mail sender MUST take action to address the described requirements in [Requirements](#requirements).

## Mailbox provider {#mailbox-provider}
An MBP MAY process the complaint and forward it to the complaint FBL address.

If the MBP wants to process the complaints and forwards it, he MUST query the Complaint-FBL-Address header.

By default, an {{!ARF=RFC5965}} compatible report MUST be sent when a manual action has been taken e.g., when a receiver marks a mail as spam, 
by clicking the "This is spam"-button in any web portal or by moving a mail to junk folder, this includes also {{?IMAP=RFC3501}} and {{?POP3=RFC1939}} movements.
The MBP SHALL NOT send any report when an automatic decisions has been made, e.g. spam filtering. 

The MBP SHOULD send a {{XARF}} compatible report, if the mail sender requests it as described in [](#xarf-report).
If this is not possible a {{!ARF=RFC5965}} compatible report MUST be sent.

The MBP MUST validate and take action to address the described requirements in [Requirements](#requirements).

# Complaint report {#complaint-report}
The complaint report (sent by MBP to mail sender) MUST be an {{!ARF=RFC5965}} report by default.
The complaint report MAY be an {{XARF}} report, if the mail sender requests it, and the MBP can send it.

The report MUST contain at least the Message-ID {{!MAIL=RFC5322}}. If present, the header "CFBL-Feedback-ID" of the complaining email MUST be added additionally.

The MBP MAY omit all further headers and/or body to comply with any data-regulation laws.

It is highly RECOMMENDED that, if used, the CFBL-Feedback-ID includes a hard to forge component such as an {{?HMAC=RFC2104}} using a secret
key, instead of a plain-text string. 

## XARF compatible report {#xarf-report}
A mail sender that wishes to receive a {{XARF}} compatible report, MUST append "report=xarf" to the [Complaint-FBL-Address header](#cfbl-address-header).
The resulting header would be the following: 

~~~
Complaint-FBL-Address: fbl@example.com; report=xarf
~~~

# Header Syntax

## Complaint-FBL-Address {#cfbl-address-header}
The following ABNF imports fields, WSP, CRLF and addr-spec from {{!MAIL=RFC5322}}.

~~~ abnf
fields /= cfbl-address

cfbl-address = "Complaint-FBL-Address:" 0*1WSP addr-spec 
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

# Security Considerations
This section discusses possible security issues, and their possible solutions, of a complaint FBL address header.

## Attacks on the FBL address
As any other email address, a complaint FBL addresses can be an attack vector for malicious emails. 
The complaint FBL addresses can be for example flooded with spam. 
This is an existing problem with any existing email address and isn't newly created by this document.

The broadcast marketing lists operator/mail sender must take appropriated measures.
One possible countermeasure would be a rate limit on the delivering IP.
However, this should be done with caution, the normal FBL email traffic must not be impaired.

## Automatic suspension of an account
Sending a FBL report against a mailbox may lead into an inaccessibility for the account owner, if there is a too quick automatic account suspension. 
For example, someone sends a invitation to his friends. For somewhat reason someone marks this mail as spam.
Now, if there is an too quick automatic account suspension in place, the senders account will be suspended, and the sender can't access his mails anymore.

MBPs and mail senders MUST take appropriate measures to prevent this.
MBPs and mail senders have therefore, mostly proprietary, ways to evaluate the trustworthiness of an account.
For example MBPs and mail senders can consider the account age and/or any other account suspension that happened beforehand, before an account gets suspended.

## Enumeration attacks / provoking unsubscription 
A malicious person can send a bunch of forged ARF reports to a known complaint FBL addresses and try to guess a Message-ID/CFBL-Feedback-ID.
He might try to do a mass-unsubscription of a complete marketing list. This is also an already existing problem with the current FBL implementation.

The receiving broadcast marketing lists operator/mail sender must take appropriated measures.

As a countermeasure it is recommended that the Message-ID and, if used, CFBL-Feedback-ID uses a hard to forge component such as an {{?HMAC=RFC2104}} using a secret
key, instead of a plain-text string, to make an enumeration attack impossible.

If it is impossible for the broadcast marketing lists operator/mail sender to use a hard to forge component, the broadcast marketing lists operator/mail sender
should take measures to avoid enumeration attacks.

## GDPR and other data-regulation laws
Providing such a header itself doesn't produce a data-regulation law problem.
The resulting ARF report, that is sent to the mail sender by the MBP, may conflict with a data-regulation law, as it may contain personal data. 

This document already addresses some parts of this problem and describes a data-regulation law safe way to send a FBL report.
As described in [](#complaint-report), the MBP may omit the complete body and/or headers and just sends the required fields.
Nevertheless, each MBP must consider on their own, if this implementation is acceptable and complies with the existing data-regulation laws.

As described in [](#complaint-report), it is also highly RECOMMENDED that the Message-ID and, if used, the CFBL-Feedback-ID 
includes a hard to forge component such as an {{?HMAC=RFC2104}} using a secret key, instead of a plain-text string.
See [](#hmac-example) for an example.

Using HMAC, or any other hard to forge component, ensures that only the mail sender has knowledge about the data.

# IANA Considerations

## Complaint-FBL-Address
The IANA is requested to register a new header field, per {{?RFC3864}}, into the "Permanent Message Header Field Names" registry:

~~~ abnf
Header field name: Complaint-FBL-Address
  
Applicable protocol: mail

Status: experimental

Author/Change controller: Jan-Philipp Benecke <jpb@cleverreach.com>

Specification document: this document
~~~

## CFBL-Feedback-ID
The IANA is requested to register a new header field, per {{?RFC3864}}, into the "Permanent Message Header Field Names" registry:

~~~ abnf
Header field name: CFBL-Feedback-ID

Applicable protocol: mail

Status: experimental

Author/Change controller: Jan-Philipp Benecke <jpb@cleverreach.com>

Specification document: this document
~~~

# Examples
For simplicity the DKIM header has been shortened, and some tags has been omitted.

## Simple
Email about the report will be generated:

~~~
Return-Path: <sender@mailer.example.com>
From: Awesome Newsletter <newsletter@example.com>
To: me@example.net
Subject: Super awesome deals for you
Complaint-FBL-Address: fbl@example.com; report=arf
Message-ID: <a37e51bf-3050-2aab-1234-543a0828d14a@mailer.example.com>
Content-Type: text/plain; charset=utf-8
DKIM-Signature: v=1; a=rsa-sha256; d=example.com;
       h=Content-Type:Subject:From:To:Message-ID:
       CFBL-Feedback-ID:Complaint-FBL-Address;

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
Source-Ip: 192.0.2.1

------=_Part_240060962_1083385345.1592993161900
Content-Type: text/rfc822; charset=UTF-8
Content-Transfer-Encoding: 7bit

Return-Path: <sender@mailer.example.com>
From: Awesome Newsletter <newsletter@example.com>
To: me@example.net
Subject: Super awesome deals for you
Complaint-FBL-Address: fbl@example.com; report=arf
Message-ID: <a37e51bf-3050-2aab-1234-543a0828d14a@mailer.example.com>
Content-Type: text/plain; charset=utf-8
DKIM-Signature: v=1; a=rsa-sha256; d=example.com;
       h=Content-Type:Subject:From:To:Message-ID:
       CFBL-Feedback-ID:Complaint-FBL-Address;

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
Complaint-FBL-Address: fbl@example.com; report=arf
Message-ID: <a37e51bf-3050-2aab-1234-543a0828d14a@mailer.example.com>
CFBL-Feedback-ID: 111:222:333:4444
Content-Type: text/plain; charset=utf-8
DKIM-Signature: v=1; a=rsa-sha256; d=example.com;
       h=Content-Type:Subject:From:To:Message-ID:
       CFBL-Feedback-ID:Complaint-FBL-Address;

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
Source-Ip: 192.0.2.1

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
Complaint-FBL-Address: fbl@example.com; report=arf
Message-ID: <a37e51bf-3050-2aab-1234-543a0828d14a@mailer.example.com>
CFBL-Feedback-ID: 3789e1ae1938aa2f0dfdfa48b20d8f8bc6c21ac34fc5023d
       63f9e64a43dfedc0
Content-Type: text/plain; charset=utf-8
DKIM-Signature: v=1; a=rsa-sha256; d=example.com;
       h=Content-Type:Subject:From:To:Message-ID:
       CFBL-Feedback-ID:Complaint-FBL-Address;

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
Source-Ip: 192.0.2.1

------=_Part_240060962_1083385345.1592993161900
Content-Type: text/rfc822-headers; charset=UTF-8
Content-Transfer-Encoding: 7bit

CFBL-Feedback-ID: 3789e1ae1938aa2f0dfdfa48b20d8f8bc6c21ac34fc5023d
       63f9e64a43dfedc0
------=_Part_240060962_1083385345.1592993161900--
~~~

# Acknowledgments
Technical and editorial reviews and comments were provided by the colleagues at CleverReach, the colleagues at Certified Senders Alliance,
Arne Allisat and Tobias Herkula (1&1 Mail & Media) and Sven Krohlas (BFK Edv-consulting).

--- back
