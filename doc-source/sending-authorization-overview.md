# Overview of Amazon SES Sending Authorization<a name="sending-authorization-overview"></a>

This topic provides an overview of the sending authorization process and then explains how the email sending features of Amazon SES, such as sending quotas and notifications, work with sending authorization\.

This section uses the following terms:
+ **Identity** – An email address or domain that Amazon SES users use to send email\.
+ **Identity owner** – An Amazon SES user who has verified ownership of an email address or domain by using the procedures described in [Verifying identities](verify-addresses-and-domains.md)\. 
+ **Delegate sender** – An entity that is authorized to send email from an identity it does not own\. An AWS account, an AWS Identity and Access Management \(IAM\) user, or an AWS service can have this *cross\-account* authority\.
+ **Sending authorization policy** – A document that you attach to an identity to specify who may send for that identity and under which conditions\.
+ **Amazon Resource Name \(ARN\)** – A standardized way to uniquely identify an AWS resource across all AWS services\. In the case of sending authorization, the resource is the identity that the identity owner wants the delegate sender to use\. An example of an ARN is *arn:aws:ses:us\-west\-2:123456789012:identity/example\.com*\. 

## Sending Authorization Process<a name="sending-authorization-process"></a>

Sending authorization is based on sending authorization policies\. If you want to enable a delegate sender to send on your behalf, you create a sending authorization policy and associate the policy to your identity by using the Amazon SES console or the Amazon SES API\. When the delegate sender attempts to send an email through Amazon SES on your behalf, the delegate sender passes the ARN of your identity in the request or in the header of the email\.

When Amazon SES receives the request to send the email, it checks your identity's policy \(if present\) to determine if you have authorized the delegate sender to send on the identity's behalf\. If the delegate sender is authorized, Amazon SES accepts the email; otherwise, Amazon SES returns an error message\.

The following diagram shows the high\-level relationship between sending authorization concepts:

![\[Sending Authorization Overview\]](http://docs.aws.amazon.com/ses/latest/DeveloperGuide/images/sending_authorization_overview.png)

The sending authorization process consists of the following steps:

1. The identity owner verifies an identity with Amazon SES by using the Amazon SES console or the Amazon SES API\. For information about the verification procedure, see [Verifying identities](verify-addresses-and-domains.md)\.

1. The delegate sender gives the identity owner the AWS account ID, IAM user ARN, or AWS service name of the entity that will do the sending\.

1. The identity owner creates a sending authorization policy and attaches the policy to the identity by using the Amazon SES console or the Amazon SES API\.

1. The identity owner gives the delegate sender the ARN of the identity so that the delegate sender can provide the ARN to Amazon SES at the time of email sending\.

1. The delegate sender sets up bounce and complaint notifications\. The identity owner can also set up email feedback notifications for bounce and complaint events\. Both the identity owner and the delegate sender can also set up [event publishing](monitor-using-event-publishing.md) to capture sending event data\.
**Note**  
If the identity owner disables sending event notifications, the delegate sender must set up event publishing to publish bounce and complaint events to an Amazon SNS topic or a Kinesis Data Firehose stream\. The sender must also apply the configuration set that contains the event publishing rule to each email they send\. If neither the identity owner nor the delegate sender sets up a method of sending notifications for bounce and complaint events, then Amazon SES automatically sends event notifications by email to the address in the Return\-Path field of the email \(or the address in the Source field, if you didn't specify a Return\-Path address\), even if the identity owner disabled email feedback forwarding\.

1. The delegate sender attempts to send an email through Amazon SES on behalf of the identity owner by passing the ARN of the identity owner's identity in the request or in the header of the email\. The delegate sender can send the email by using either the Amazon SES SMTP interface or the Amazon SES API\. Upon receiving the request, Amazon SES examines any policies that are attached to the identity, and accepts the email if the delegate sender is authorized to use the specified "From" address and "Return Path" address; otherwise, Amazon SES returns an error and does not accept the message\.

1. If the identity owner needs to de\-authorize the delegate sender, the identity owner edits the sending authorization policy or deletes the policy entirely\. The identity owner can perform either action by using the Amazon SES console or the Amazon SES API\. 

For more information about how the identity owner or delegate sender perform those tasks, see [Identity Owner Tasks](sending-authorization-identity-owner-tasks.md) or [Delegate Sender Tasks](sending-authorization-delegate-sender-tasks.md), respectively\.

## Attribution of Email Sending Features<a name="sending-authorization-attribution"></a>

It is important to understand the role of the delegate sender and the identity owner with respect to Amazon SES email sending features such as daily sending quota, bounces and complaints, DKIM signing, feedback forwarding, and so on\. The attribution is the following:
+ **Sending quotas** – Email sent from the identity owner's identities count against the delegate sender's quotas\.
+ **Bounces and complaints** – Bounce and complaint events are recorded against the delegate sender's Amazon SES account, and can therefore impact the delegate sender's reputation\.
+ **DKIM signing** – If the identity owner has enabled Easy DKIM signing for an identity, all email sent from that identity will be DKIM\-signed, including email sent by the delegate sender\. Only the identity owner can control whether the emails are DKIM\-signed\.
+ **Notifications** – Both the identity owner and the delegate sender can set up notifications for bounces and complaints\. The email identity owner can also enable email feedback forwarding\. For information about setting up notifications, see [Monitoring your Amazon SES sending activity](monitor-sending-activity.md)\.
+ **Verification** – Identity owners are responsible for following the procedure in [Verifying identities](verify-addresses-and-domains.md) to verify that they own the email addresses and domains that they are authorizing delegate senders to use\. Delegate senders do not need to verify any email addresses or domains specifically for sending authorization\.
+ **AWS Regions** – The delegate sender must send the emails from the AWS Region in which the identity owner's identity is verified\. The sending authorization policy that gives permission to the delegate sender must be attached to the identity in that region\.
+ **Billing** – All messages that are sent from the delegate sender's account, including emails that the delegate sender sends using the identity owner's addresses, are billed to the delegate sender\. 
