# Perfect – SMTP


This project provides an SMTP library.

This package builds with Swift Package Manager and is part of the [Perfect](https://github.com/PerfectlySoft/Perfect) project. Ensure you have installed and activated the latest Swift 4.0 tool chain.

## Linux Build Note

Make sure libssl-dev was installed on Ubuntu 16.04:

```
sudo apt-get install libssl-dev
```

## Relevant Examples

* [Perfect-SMTP-Demo](https://github.com/PerfectExamples/Perfect-SMTP-Demo)


## Quick Start

To use SMTP class, modify the Package.swift file and add following dependency:

``` swift
.Package(url: "https://github.com/PerfectlySoft/Perfect-SMTP.git", majorVersion: 3)
```

Then import SMTP library into the Swift source code:

``` swift
import PerfectSMTP
```

## Data Structures

Perfect SMTP contains three different data structures: SMTPClient, Recipient and EMail.

### SMTPClient

SMTPClient object is a data structure to store mail server login information:

``` swift
let client = SMTPClient(url: "smtp://mailserver.address", username: "someone@some.where", password:"secret")
```

### Recipient

Recipient object is a data structure which store one's name and email address:

``` swift
let recipient = Recipient(name: "Someone's Full Name", address: "someone@some.where")
```

### EMail

Use email object to compose and send an email. Check the following example code:

``` swift
// initialize an email draft with mail connection / login info
var email = EMail(client: client)

// set the title of email
email.subject = "Mail Title"

// set the sender info
email.from = Recipient(name: "My Full Name", address: "mynickname@my.home")

// fill in the main content of email, plain text or html
email.html = "<h1>Hello, world!</h1><hr><img src='http://www.perfect.org/images/perfect-logo-2-0.svg'>"

// set the mail recipients, to / cc / bcc are all arrays
email.to.append(Recipient(name: "First Receiver", address: "someone@some.where"))
email.cc.append(Recipient(name: "Second Receiver", address: "someOtherOne@some.where"))
email.bcc.append(Recipient(name: "An invisible receiver", address: "someoneElse@some.where"))

// add attachments
email.attachments.append("/path/to/file.txt")
email.attachments.append("/path/to/img.jpg")

// send the email and call back if done.
do {
  try email.send { code, header, body in
    /// response info from mail server
    print(code)
    print(header)
    print(body)
  }//end send
}catch(let err) {
  /// something wrong
}
```

#### Members of EMail Object

- client: SMTPClient, login info for mail server connection
- to: [Recipient], array of mail recipients
- cc: [Recipient], array of mail recipients, "copy / forward"
- bcc:[Recipient], array of mail recipients, will not appear in the to / cc mail.
- from: Recipient, email address of the current sender
- subject: String, title of the email
- attachments: [String], full path of attachments, i.e., ["/path/to/file1.txt", "/path/to/file2.gif" ...]
- content: String, mail body in text, plain text or html
- html: String, alias of `content` (shares the same variable as `content`)
- text: String, set the content to plain text
- send(completion: @escaping ((Int, String, String)->Void)), function of sending email with callback. The completion callback has three parameters; check Perfect-CURL `performFully()` for more information:
  - code: Int, mail server response code. Zero for OK.
  - header: String, mail server response header string.
  - body: String, mail server response body string.

## Summary

For better understanding, here is the brief structure of sending emails:

``` swift
import PerfectSMTP

let client = SMTPClient(url: "smtp://smtp.gmx.com", username: "yourname@youraddress.com", password:"yourpassword")

var email = EMail(client: client)

email.subject = "a topic"
email.content = "a message"

email.cc.append(Recipient(address: "who@where.com"))

do {
  try email.send { code, header, body in
    /// response info from mail server
    print(code)
  }//end send
}catch(let err) {
  /// something wrong
}
```

## Example

A demo can be found here:
[Perfect SMTP Demo](https://github.com/PerfectExamples/Perfect-SMTP-Demo)

## Tips for SMTPS

We've received a lot of requests about google smtp examples, Thanks for @ucotta @james and of course the official Perfect support from @iamjono, this note might be helpful for building gmail applications: ⚠️*the SMTPClient url needs to be `smtps://smtp.gmail.com:465`, and you may need to “turn on access for less secure apps” in the google settings.*⚠️

Please check the SMTPS code below, note the only difference is the URL pattern:

``` swift
import PerfectSMTP

let client = SMTPClient(url: "smtps://smtp.gmail.com:465", username: "yourname@gmail.com", password:"yourpassword")

var email = EMail(client: client)

email.subject = "a topic"
email.content = "a message"

email.cc.append(Recipient(address: "who@where.com"))

do {
  try email.send { code, header, body in
    /// response info from mail server
    print(code)
  }//end send
}catch(let err) {
  /// something wrong
}
```
