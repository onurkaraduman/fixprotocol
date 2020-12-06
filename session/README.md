FIX Session Layer Test Cases

**This document has been copied from https://www.fixtrading.org/standards/fix-session-testcases-online/** 


FIX session layer test cases
============================

Applicability
-------------

This document was last revised September 20, 2002 at which time FIX version 4.3 with Errata 20020930 was the latest version of the FIX Protocol. Note that future amendments to this document may be found on the FIX website and any version of this document published on a later date takes precedence over this version of the document. This document is applicable to all supported versions of the FIX session layer (4.2, 4.4, FIXT) except where explicitly indicated.

Test cases
----------

These test cases are from the perspective of the FIX system being tested. The FIX system receives the “Condition / Stimulus” and is expected to take the appropriate action as defined by “Expected Behavior”.

Buyside-oriented (session initiator) Logon and session initiation test case
---------------------------------------------------------------------------

### Scenario 1B Connect and Send Logon message

**Mandatory**

  

**Condition/Stimulus**

**Expected Behavior**

a. Establish transport layer connection.

Successfully established transport layer connection with peer.

b. Send Logon(35=A) request message.

Logon(35=A) acknowledgement message sent by peer.

c. Valid Logon(35=A) acknowledgement message received.

If MsgSeqNum(34) is too high, send ResendRequest(35=2).

d. Invalid Logon(35=A) acknowledgement message received.

1.  Generate an error condition in test output.
2.  (Optional) Send Reject(35=3) message with RefSeqNum(45) identifying Logon(35=A) message’s MsgSeqNum(34) and Text(58) referencing error condition.
3.  Send Logout(35=5) message with Text(58) referencing error condition.
4.  Disconnect.

e. Receive any message other than a Logon(35=A) message.

1.  Log an error “first message not a logon”
2.  (Optional) Send Reject(35=3) message with RefSeqNum(45) identifying message’s MsgSeqNum(34) and Text(58) referencing error condition.
3.  (Optional) Send Logout(35=5) message with Text(58) referenceing error condition.
4.  Disconnect.

Sellside-oriented (session acceptor) Logon and session initiation test case
---------------------------------------------------------------------------

### Scenario 1S Receive Logon message

**Mandatory**

  

**Condition/Stimulus**

**Expected Behavior**

a. Valid Logon(35=A) request message received.

1.  Respond with Logon(35=A) acknowledgement message.
2.  If MsgSeqNum(34) > NextNumIn send ResendRequest(35=2).

b. Logon(35=A) message received with duplicate identity (e.g. same IP, port, SenderCompID(49), TargetCompID(56), etc. as existing connection).

1.  Generate an error condition in test output.
2.  Disconnect without sending a message (Note: sending a Reject or Logout(35=5) would consume a MsgSeqNum(34)).

c. Logon(35=A) message received with unauthenticated/non-configured identity (e.g. invalid SenderCompID(49), invalid TargetCompID(56), invalid source IP address, etc. vs. system configuration).

1.  Generate an error condition in test output.
2.  Disconnect without sending a message (Note: sending a Reject or Logout(35=5) would consume a MsgSeqNum(34)).

d. Invalid Logon(35=A) message.

1.  Generate an error condition in test output.
2.  (Optional) Send Reject(35=3) message with RefSeqNum(45) identifying Logon(35=A) message’s MsgSeqNum(34) with Text(58) referencing error condition.
3.  Send Logout(35=5) message with Text(58) referencing error condition.
4.  Disconnect.

### Scenario 2S. Receive any message other than a Logon message

**Mandatory**

  

**Condition/Stimulus**

**Expected Behavior**

First message received is not a Logon(35=A) message.

1.  Log an error “First message not a logon”.
2.  Disconnect.

Test cases applicable to all FIX systems
----------------------------------------

### Scenario 2 Receive Message Standard Header

**Mandatory**

  

**Condition/Stimulus**

**Expected Behavior**

a. MsgSeqNum(34) received as expected.

Accept MsgSeqNum(34) for the message.

b. MsgSeqNum(34) higher than expected.

Respond with ResendRequest(35=2) message.

c. MsgSeqNum(34) lower than expected without PossDupFlag(43) set to Y.

Exception: SequenceReset(35=4).

1.  Whenever possible it is recommended that FIX engine attempt to send a Logout(35=5) message with a text message of “MsgSeqNum too low, expecting X but received Y”.
2.  (Optional) Wait for Logout(35=5) message response (Note: likely will have inaccurate MsgSeqNum(34)) or wait 2 seconds, whichever comes first.
3.  Disconnect.
4.  Generate an error condition in test output.

d. Garbled message received.

1.  Consider garbled and ignore message (do not increment NextNumIn) and continue accepting messages.
2.  Generate a warning condition in test output.

e. PossDupFlag(43) set to Y; OrigSendingTime(122) specified is less than or equal to SendingTime(52) and MsgSeqNum(34) lower than expected.

Note: OrigSendingTime(122) should be earlier than SendingTime(52) unless the message is being resent within the same second during which it was sent.

1.  Check to see if MsgSeqNum(34) has already been received.
2.  If already received then ignore the message, otherwise accept and process the message.

f. PossDupFlag(43) set to Y; OrigSendingTime(122) specified is greater than SendingTime(52) and MsgSeqNum(34) as expected.

Note: OrigSendingTime(122) should be earlier than SendingTime(52) unless the message is being resent within the same second during which it was sent.

1.  Send Reject(35=3) message with SessionRejectReason(373) set to 10 (SendingTime accuracy problem).
2.  Increment NextNumIn.
3.  Optional:

*   Send Logout(35=5) message referencing inaccurate SendingTime(52) value.
*   (Optional) Wait for Logout(35=5) message response (Note: likely will have inaccurate SendingTime(52)) or wait 2 seconds, whichever comes first.
*   Disconnect.
*   Generate an error condition in test output.

g. PossDupFlag(43) set to Y and OrigSendingTime(122) not specified.

Note: Always set OrigSendingTime(122) to the time when the message was originally sent-not the present SendingTime(52) and set PossDupFlag(43)=Y when responding to a ResendRequest(35=2).

1.  Send Reject(35=3) message with SessionRejectReason(373) set to 1 (Required tag missing).
2.  Increment NextNumIn.

h. BeginString(8) value received as expected and specified in testing profile and matches BeginString(8) on outbound messages.

Accept BeginString(8) for the message.

i. BeginString(8) value received did not match value expected and specified in testing profile or does not match BeginString(8) on outbound messages.

1.  Send Logout(35=5) message referencing incorrect BeginString(8) value.
2.  (Optional) Wait for Logout(35=5) message response (Note: likely will have incorrect BeginString(8)) or wait _LogoutAckThreshold_ seconds, whichever comes first.
3.  Disconnect.
4.  Generate an error condition in test output.

j. SenderCompID(49) and TargetCompID(56) values received as expected and specified in testing profile.

Accept SenderCompID(49) and TargetCompID(56) for the message.

k. SenderCompID(49) and TargetCompID(56) values received did not match values expected and specified in testing profile.

1.  Send Reject(35=3) message with RefTagID(371) to identify field with mismatched value, and SessionRejectReason(373) set to 9 (CompID problem).
2.  Increment NextNumIn.
3.  Send Logout(35=5) message referencing incorrect SenderCompID(49) or TargetCompID(56) value.
4.  (Optional) Wait for Logout(35=5) message response (Note: likely will have incorrect SenderCompID(49) or TargetCompID(56)) or wait 2 seconds, whichever comes first.
5.  Disconnect.
6.  Generate an error condition in test output.

l. BodyLength(9) value received is correct.

Accept BodyLength(9) for the message.

m. BodyLength(9) value received is not correct.

1.  Consider garbled and ignore message (do not increment NextNumIn) and continue accepting messages.
2.  Generate a warning condition in test output.

n. SendingTime(52) value received is specified in UTC (Universal Time Coordinated) also known as GMT) or is not within _SendingTimeThreshold_ seconds of a synchronized time source.

Accept SendingTime(52) for the message.

o. SendingTime(52) value received is either not specified in UTC (Universal Time Coordinated) or is not GMT) or is not within a within _SendingTimeThreshold_ seconds of a synchronized time source.

Rationale:

Verify system clocks on both sides are in sync and that SendingTime(52) is current time.

1.  Send Reject(35=3) message with SessionRejectReason(373) set to 10 (SendingTime accuracy problem).
2.  Increment NextNumIn.
3.  Send Logout(35=5) message referencing inaccurate SendingTime(52) value.
4.  (Optional) Wait for Logout(35=5) message response (Note: likely will have inaccurate SendingTime(52)) or wait 2 seconds, whichever comes first.
5.  Disconnect.
6.  Generate an error condition in test output.

p. MsgType(35) value received is valid (defined in spec or classified as user-defined).

Accept MsgType(35) for the message.

q. MsgType(35) value received is not valid (defined in spec or classified as user-defined).

1.  Send Reject(35=3) message with SessionRejectReason(373) set to 11 (Invalid MsgType).
2.  Increment NextNumIn.
3.  Generate a warning condition in test output.

r. MsgType(35) value received is valid (defined in spec or classified as user-defined) but not supported or registered in testing profile.

1.  Send BusinessMessageReject(35=j) with BusinessRejectReason(380) set to 3 (Unsupported Message Type).
2.  Increment NextNumIn.
3.  Generate a warning condition in test output.

s. BeginString(8), BodyLength(9), and MsgType(35) are first three fields of message.

Accept the message.

t. BeginString(8), BodyLength(9), and MsgType(35) are not the first three fields of message.

1.  Consider garbled and ignore message.
2.  Do not increment NextNumIn.
3.  Continue accepting messages.
4.  Generate a warning condition in test output.

### Scenario 3 Receive Message Standard Trailer

**Mandatory**

  

**Condition/Stimulus**

**Expected Behavior**

a. Valid CheckSum(10).

Accept message

b. Invalid CheckSum(10).

1.  Consider garbled and ignore message.
2.  Do not increment NextNumIn.
3.  Continue accepting messages.
4.  Generate a warning condition in test output.

c. Garbled message.

1.  Consider garbled and ignore message.
2.  Do not increment NextNumIn.
3.  Continue accepting messages.
4.  Generate a warning condition in test output.

d. CheckSum(10) is last field of message, value has length of 3, and is delimited by `<SOH>`.

Accept message.

e. CheckSum(10) is not the last field of message, value does not have length of 3, or is not delimited by `<SOH>`.

1.  Consider garbled and ignore message.
2.  Do not increment NextNumIn.
3.  Continue accepting messages.
4.  Generate a warning condition in test output.

### Scenario 4 Send Heartbeat message

**Mandatory**

  

**Condition/Stimulus**

**Expected Behavior**

a. No data sent during preset heartbeat interval (HeartBtInt(108) field).

Send Heartbeat(35=0) message.

b. TestRequest(35=1) message received.

Send Heartbeat(35=0) messsage with TestRequest(35=1) message’s TestReqID(112).

### Scenario 5 Receive Heartbeat message

**Mandatory**

  

**Condition/Stimulus**

**Expected Behavior**

Valid Heartbeat(35=0) message.

Accept Heartbeat(35=0) message.

### Scenario 6 Send Test Request

**Mandatory**

  

**Condition/Stimulus**

**Expected Behavior**

No data received during preset heartbeat interval (HeartBtInt(108)) + “some reasonable period of time” (use 20% of HeartBtInt(108)).

1.  Send TestRequest(35=1) message.
2.  Track and verify that a Heartbeat(35=0) message with the same TestReqID(112) is received (may not be the next message received).

### Scenario 7 Receive Reject message

**Mandatory**

  

**Condition/Stimulus**

**Expected Behavior**

Valid Reject(35=3) message.

1.  Increment NextNumIn.
2.  Continue accepting messages.

### Scenario 8 Receive Resend Request message

**Mandatory**

  

**Condition/Stimulus**

**Expected Behavior**

Valid ResendRequest(35=2)

Respond with application layer messages and SequenceReset(35=4) with GapFillFlag(123)=Y for session layer messages in request range according to message recovery rules.

### Scenario 9 Synchronize sequence numbers

**Optional**

  

**Condition/Stimulus**

**Expected Behavior**

Application failure that caused loss of session state.

1.  Establish new FIX connection.
2.  Reset NextNumOut to an arbitrarily large value that is greater than the last known NextNumOut value.
3.  Reset NextNumIn value to 1.
4.  Send SequenceReset(35=4) with GapFillFlag(123)=N and NewSeqNo(36) set to a number arbitrarily larger than the last known NextNumOut value.
5.  Process retransmitted messages and gap fills from peer.

**OR**

1.  Both peers reset their NextNumIn and NextNumOut values to 1.
2.  Start new FIX session

### Scenario 10 Receive Sequence Reset (Gap Fill)

**Mandatory**

  

**Condition/Stimulus**

**Expected Behavior**

a. Receive SequenceReset(35=4) with GapFillFlag(123)=Y message with NewSeqNo(36) > MsgSeqNum(34)

and

MsgSeqNum(34) > NextNumIn

Issue ResendRequest(35=2) to fill gap between last expected MsgSeqNum(34) & received MsgSeqNum(34).

b. Receive SequenceReset(35=4) with GapFillFlag(123)=Y message with NewSeqNo(36) > MsgSeqNum(34)

and

MsgSeqNum(34) = NextNumIn

Set next expected sequence number = NewSeqNo(36).

c. Receive SequenceReset(35=4) with GapFillFlag(123)=Y message with NewSeqNo(36) > MsgSeqNum(34)

and

MsgSeqNum(34) < NextNumIn

and

PossDupFlag(43)=Y.

Ignore message.

d. Receive SequenceReset(35=4) with GapFillFlag(123)=Y message with NewSeqNo(36) > MsgSeqNum(34)

and

MsgSeqNum(34) < NextNumIn

and

without PossDupFlag(43)=Y.

1.  If possible, send a Logout(35=5) message with text of “MsgSeqNum too low, expecting X received Y,” prior to disconnecting FIX session.
2.  (Optional) Wait for Logout(35=5) message response (Note: likely will have inaccurate MsgSeqNum(34)) or wait 2 seconds, whichever comes first.
3.  Disconnect.
4.  Generate an error condition in test output.

e. Receive SequenceReset(35=4) with GapFillFlag(123)=Y message with NewSeqNo(36) <= MsgSeqNum(34)

and

MsgSeqNum(34) = NextNumIn

Send Reject(35=3) message with message “attempt to lower sequence number, invalid value NewSeqNo(36)=<x>”.

### Scenario 11 Receive Sequence Reset (Reset)

**Mandatory**

  

**Condition/Stimulus**

**Expected Behavior**

a. Receive SequenceReset(35=4) with GapFillFlag(123)=N message with NewSeqNo(36) > NextNumIn

1.  Accept the SequenceReset(35=4) message without regard to its MsgSeqNum(34).
2.  Set NextNumIn to NewSeqNo(36).

b. Receive SequenceReset(35=4) with GapFillFlag(123)=N message with NewSeqNo(36) = NextNumIn

1.  Accept the SequenceReset(35=4) message without regard to its MsgSeqNum(34).
2.  Generate a warning condition in test output.

c. Receive SequenceReset(35=4) with GapFillFlag(123)=N message with NewSeqNo(36) < NextNumIn

1.  Accept the SequenceReset(35=4) message without regard to its MsgSeqNum(34).
2.  Send Reject(35=3) message with SessionRejectReason(373) set to 5 (Value is incorrect (out of range) for this tag).
3.  Do NOT change the value of NextNumIn.
4.  Generate an error condition in test output.

### Scenario 12 Initiate logout process

**Mandatory**

  

**Condition/Stimulus**

**Expected Behavior**

Initiate logout.

1.  Send Logout(35=5) message.
2.  Wait for counterparty to respond with Logout(35=5) message _LogoutAckThreshold_ seconds (Note: may not be received if communications problem exists). If not received, generate a warning condition in test output.
3.  Disconnect.

### Scenario 13 Receive Logout message

**Mandatory**

  

**Condition/Stimulus**

**Expected Behavior**

a. Receive valid Logout(35=5) acknowledgement in response to a Logout(35=5) request.

Disconnect without sending a message.

b. Receive valid Logout(35=5) request message unsolicited.

1.  Send Logout(35=5) acknowledgement message.
2.  Wait for counterparty to disconnect up to 10 seconds. If max exceeded, disconnect and generate an error condition in test output.

### Scenario 14 Receive application or session layer message

**Mandatory**

  

**Condition/Stimulus**

**Expected Behavior**

a. Receive field identifier (tag number) not defined in specification.

Exception: undefined tag used is specified in testing profile as user-defined.

1.  Send Reject(35=3) message with SessionRejectReason(373) set to 0 (Invalid tag number).
2.  Increment NextNumIn.
3.  Generate an error condition in test output.

b. Receive message with a required field identifier (tag number) missing.

1.  Send Reject(35=3) message with SessionRejectReason(373) set to 1 (Required tag missing).
2.  Increment NextNumIn.
3.  Generate an error condition in test output.

c. Receive message with field identifier (tag number) which is defined in the specification but not defined for this message type.

Exception: undefined tag used is specified in testing profile as user-defined for this message type.

1.  Send Reject(35=3) message with SessionRejectReason(373) set to 2 (Tag not defined for this message type).
2.  Increment NextNumIn.
3.  Generate an error condition in test output.

d. Receive message with field identifier (tag number) specified but no value (e.g. “55=`<SOH>`” vs. “55=IBM`<SOH>`”).

1.  Send Reject(35=3) message with SessionRejectReason(373) set to 4 (Tag specified without a value).
2.  Increment NextNumIn.
3.  Generate an error condition in test output.

e. Receive message with incorrect value (out of range or not part of valid list of enumerated values) for a particular field identifier (tag number).

Exception: undefined enumeration values used are specified in testing profile as user-defined.

1.  Send Reject(35=3) message with SessionRejectReason(373) set to 5 (Value is incorrect (out of range) for this tag).
2.  Increment inbound MsgSeqNum(34).
3.  Generate an error condition in test output.

f. Receive message with a value in an incorrect data format (syntax) for a particular field identifier (tag number).

1.  Send Reject(35=3) message with SessionRejectReason(373) set to 6 (Incorrect data format for value).
2.  Increment NextNumIn.
3.  Generate an error condition in test output.

g. Receive a message in which the following is not true: Standard Header fields appear before Body fields which appear before Standard Trailer fields.

1.  Send Reject(35=3) message with SessionRejectReason(373) set to 14 (Tag specified out of required order).
2.  Increment NextNumIn.
3.  Generate an error condition in test output.

h. Receive a message in which a field identifier (tag number) which is not part of a repeating group is specified more than once.

1.  Send Reject(35=3) message with SessionRejectReason(373) set to 13 (Tag appears more than once).
2.  Increment NextNumIn.
3.  Generate an error condition in test output.

i. Receive a message with repeating groups in which the “count” field value for a repeating group is incorrect.

1.  Send Reject(35=3) message with SessionRejectReason(373) set to 16 (Incorrect NumInGroup count for repeating group).
2.  Increment NextNumIn.
3.  Generate an error condition in test output.

j. Receive a message with repeating groups in which the order of repeating group fields does not match the specification.

1.  Send Reject(35=3) message with SessionRejectReason(373) set to 15 (Repeating group fields out of order).
2.  Increment NextNumIn
3.  Generate an error condition in test output.

k. Receive a message with a field of a data type other than “data” which contains one or more embedded `<SOH>` values.

1.  Send Reject(35=3) message with SessionRejectReason(373) (Non “data” value includes field delimiter (`<SOH>` character)).
2.  Increment NextNumIn
3.  Generate an error condition in test output.

**OR**

1.  Consider garbled and ignore message.
2.  Generate an error condition in test output

l. Receive a message when application layer processing or system is not available (optional).

1.  Send BusinessMessageReject(35=j) with BusinessRejectReason(380) set to 4 (Application not available).
2.  Increment NextNumIn.
3.  Generate a warning condition in test output.

m. Receive a message in which a conditionally required field is missing.

1.  Send BusinessMessageReject(35=j) with BusinessRejectReason(380) set to 5 (Conditionally Required Field Missing).
2.  Increment NextNumIn.
3.  Generate an error condition in test output.

n. Receive a message in which a field identifier (tag number) appears in both cleartext and encrypted section but has different values.

1.  Send Reject(35=3) message with SessionRejectReason(373) set to 7 (Decryption problem).
2.  Increment NextNumIn.
3.  Generate an error condition in test output.

### Scenario 15 Send application or session layer messages to test normal and abnormal behavior/response

**Optional**

  

**Condition/Stimulus**

**Expected Behavior**

Send more than one message of the same type with header and body fields ordered differently to verify acceptance. (Exclude those which have restrictions regarding order.)

Message accepted and subsequent messages’ MsgSeqNum(34) are accepted.

### Scenario 16 Queue outgoing messages

**Mandatory**

  

**Condition/Stimulus**

**Expected Behavior**

a. Message to send/queue while disconnected.

Queue outgoing messages. Note there are two valid approaches:

1.  Queue without regard to MsgSeqNum(34).
    1.  Store data for messages
2.  Queue each message with the next MsgSeqNum(34) value.
    2.  Store data for messages in such a manner as to use and “consume” the next MsgSeqNum(34).

Note: SendingTime(52) must contain the time the message is sent, not the time the message was queued.

b. Reconnect with queued messages.

1.  Complete logon process (establish transport layer connection and exchange Logon(35=A) messages).
2.  Synchronize the FIX session, if required (inbound Logon(35=A) message MsgSeqNum(34) > NextNumIn).
3.  Recommended short delay, or use TestRequest(35=1) or Heartbeat(35=0) message to verify FIX session synchronization completed.
4.  Note there are two valid queuing approaches.
    1.  Queue without regard to MsgSeqNum(34):
        1.  Send queued messages with new MsgSeqNum(34) values (greater than Logon(35=A) message’s MsgSeqNum(34)).
    2.  Queue each message with the next MsgSeqNum(34) value:
        2.  (Note: Logon(35=A) message’s MsgSeqNum(34) will be greater than the queued messages’ MsgSeqNum(34)).
        3.  Counterparty will issue ResendRequest(35=2) requesting the range of missed messages.
        4.  Resend each queued message with PossDupFlag(43) set to Y.

Note: SendingTime(52) must contain the time the message is sent, not the time the message was queued.

### Scenario 17 Support encryption

**Optional**

  

**Condition/Stimulus**

**Expected Behavior**

a. Receive Logon(35=A) request message with valid, supported EncryptMethod(98).

1.  Accept the message.
2.  Perform the appropriate decryption and encryption method readiness.
3.  Respond with Logon(35=A) acknowledgement message with the same EncryptMethod(98).

b. Receive Logon(35=A) message with invalid or unsupported EncryptMethod(98).

1.  Send Reject(35=3) message with SessionRejectReason(373) set to 7 (Decryption problem).
2.  Increment NextNumIn.
3.  Send Logout(35=5) request message referencing invalid or unsupported EncryptMethod(98) value.
4.  (Optional) Wait for Logout(35=5) acknowledgement message response (Note: could have decrypt problems) or wait _LogoutAckThreshold_ seconds, whichever comes first.
5.  Disconnect.
6.  Generate an error condition in test output.

c. Receive message with valid SignatureLength(93) and Signature(89) values.

Accept the message.

d. Receive message with invalid SignatureLength(93) value.

1.  Send Reject(35=3) message with SessionRejectReason(373) set to 8 (Signature problem).
2.  Increment inbound MsgSeqNum(34).
3.  Generate an error condition in test output.

e. Receive message with invalid Signature(89) value.

1.  Send Reject(35=3) message with SessionRejectReason(373) set to 8 (Signature problem).
2.  Increment inbound MsgSeqNum(34).
3.  Generate an error condition in test output.

Or consider decryption error or message out of order, ignore message (do not increment inbound MsgSeqNum(34)) and continue accepting messages.

f. Receive message with a valid SecureDataLen(90) value and a SecureData(91) value that can be decrypted into valid, parsable cleartext.

Accept the message.

g. Receive message with invalid SecureDataLen(90) value.

1.  Consider garbled and ignore message.
2.  Generate an error condition in test output

h. Receive message with a SecureData(91) value that cannot be decrypted into valid, parsable cleartext.

1.  Send Reject(35=3) message with SessionRejectReason(373) set to 7 (Decryption problem).
2.  Increment NextNumIn.
3.  Generate an error condition in test output.

i. Receive message with one or more fields not present in the unencrypted portion of the message that “must be unencrypted” according to the spec.

1.  Send Reject(35=3) message with SessionRejectReason(373) set to 7 (Decryption problem).
2.  Increment NextNumIn.
3.  Generate an error condition in test output.

j. Receive message with incorrect handling of “leftover” characters (e.g. when length of cleartext prior to encryption is not a multiple of 8) according to the specified EncryptMethod(98).[1](https://www.fixtrading.org/standards/fix-session-testcases-online/#fn1)

1.  Send Reject(35=3) message with SessionRejectReason(373) set to 7 (Decryption problem).
2.  Increment NextNumIn.
3.  Generate an error condition in test output.

### Scenario 18 Support third party addressing

**Optional**

  

**Condition/Stimulus**

**Expected Behavior**

a. Receive messages with OnBehalfOfCompID(115) and

DeliverToCompID(128) values expected as specified in testing profile and with correct usage.

Accept messages.

b. Receive messages with OnBehalfOfCompID(115) or

DeliverToCompID(128) values not specified in testing profile or incorrect usage.

1.  Send Reject(35=3) message with RefTagID(371) to identify field with incorrect values, and SessionRejectReason(373) set to 9 (CompID problem).
2.  Increment NextNumIn.
3.  Generate an error condition in test output.

### Scenario 19 Test PossResend handling

**Mandatory**

  

**Condition/Stimulus**

**Expected Behavior**

a. Receive message with PossResend(97)=Y and application layer check of message-specific ID indicates that it has already been seen on this session.

1.  Ignore the message.
2.  Generate a warning condition in test output.

b. Receive message with PossResend(97)=Y and application layer check of message-specific ID indicates that it has NOT yet been seen on this session.

Accept and process the message normally.

### Scenario 20 Simultaneous Resend request test

**Mandatory**

  

**Condition/Stimulus**

**Expected Behavior**

Receive a ResendRequest(35=2) message while having sent and awaiting complete set of responses to a ResendRequest(35=2) message.

1.  Perform resend of requested messages.
2.  Send ResendRequest(35=2) message to request missed messages if gap still exists.

* * *

1.  For block ciphers which transform a fixed-size block of data (usually 64 bits) into another fixed-size block (possibly 64 bits long again) using a function selected by the key.[↩︎](https://www.fixtrading.org/standards/fix-session-testcases-online/#fnref1)

[](javascript:void(0);)

*   [Contact Us](https://www.fixtrading.org/contact-us/)
*   [Privacy Policy](https://www.fixtrading.org/privacy-policy/)
*   [Terms and Conditions](https://www.fixtrading.org/terms-and-conditions/)
*   [Advertising](https://www.fixtrading.org/advertising-opportunities/)
*   [Sitemap](https://www.fixtrading.org/sitemap/)

FIX Protocol Ltd © 2020 | All Rights Reserved


# References
* https://www.fixtrading.org/standards/fix-session-testcases-online/
