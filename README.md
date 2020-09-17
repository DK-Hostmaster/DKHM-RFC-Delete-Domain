![DK Hostmaster Logo](https://www.dk-hostmaster.dk/sites/default/files/dk-logo_0.png)

# DKHM RFC for Delete Domain EPP Command

![Markdownlint Action](https://github.com/DK-Hostmaster/DKHM-RFC-Delete-Domain/workflows/Markdownlint%20Action/badge.svg)
![Spellcheck Action](https://github.com/DK-Hostmaster/DKHM-RFC-Delete-Domain/workflows/Spellcheck%20Action/badge.svg)

2020-09-17
Revision: 1.2

## Table of Contents

<!-- MarkdownTOC bracket=round levels="1,2,3,4" indent="  " autolink="true" autoanchor="true" -->

- [Introduction](#introduction)
  - [About this Document](#about-this-document)
  - [License](#license)
  - [Document History](#document-history)
  - [XML and XSD Examples](#xml-and-xsd-examples)
- [Description](#description)
  - [Delete and Scheduling Deletion](#delete-and-scheduling-deletion)
  - [Restore](#restore)
- [XSD Definition](#xsd-definition)
- [References](#references)

<!-- /MarkdownTOC -->

<a id="introduction"></a>
## Introduction

This is a draft and proposal for changes to the process for domain name deletion via the DK Hostmaster EPP portal/service. The specification briefly touches on the registrar portal service, which mimicks the EPP service for consistency.

The overall [description of the concept][CONCEPT] of the registrar model offered by DK Hostmaster A/S provided as a general overview, where this RFC digs into the details of the cancellation/deletion of domain names in the context of an implementation proposal.

<a id="about-this-document"></a>
### About this Document

We have adopted the term RFC (_Request For Comments_), due to the recognition in the term and concept, so this document is a process supporting document, aiming to serve the purpose of obtaining a common understanding of the proposed implementation and to foster discussion on the details of the implementation. The final specification will be lifted into the [DK Hostmaster EPP Service Specification][DKHMEPPSPEC] implementation and this document will be closed for comments and the document no longer be updated.

<a id="license"></a>
### License

This document is copyright by DK Hostmaster A/S and is licensed under the MIT License, please see the separate LICENSE file for details.

<a id="document-history"></a>
### Document History

- 1.2 2020-09-17
  - Added information on restore as described in [RFC:3915][RFC3915] and [RFC:8748][RFC8748]

- 1.1 2020-09-01
  - Clarification on deletion and handling of subordinates, as specified in [RFC:5731][RFC5731]

- 1.0 2020-08-25
  - Initial revision

<a id="xml-and-xsd-examples"></a>
### XML and XSD Examples

All example XML files are available in the [DK Hostmaster EPP XSD repository][DKHMXSDSPEC].

The proposed extensions and XSD definitions are available in the  [3.2 candidate][DKHMXSD3.2] of the DK Hostmaster XSD, which is currently a draft and work in progress and marked as a  _pre-release_.

<a id="description"></a>
## Description

<a id="delete-and-scheduling-deletion"></a>
### Delete and Scheduling Deletion

In addition to the standard EPP `delete domain` command, DK Hostmaster will support scheduling of deletion of domain names, by providing a date to the EPP `delete domain` command via an optional extension.

The default is to deactivate immediately if possible, which complies with [RFC:5731][RFC5731]. Not being able to complete the request will result in a error, also in compliance with [RFC:5731][RFC5731]. Please see below for more information on the business process for deletion.

The extension offers the ability to specify a date, this date will have to be in the future and prior to, or on the expiration date of the specified domain name.

The current expiration date can be obtained using the `info domain` command and is specified in the `domain:exDate` field. The date conforms with the required format.

An example (do note the dates in the below examples are examples and are fabricated and might not be correct, meaning they might be in the past by the time of reading):

```xml
  <extension>
    <dkhm:delDate xmlns:dkhm="urn:dkhm:xml:ns:dkhm-3.2">2021-01-31T00:00:00.0Z</dkhm:delDate>
  </extension>
```

The date follows the format used in the EPP protocol, which complies with [RFC:3339][RFC3339].

The XSD for the extension look as follows:

```xsd
  <!-- custom: delDate  -->
  <simpleType name="delDate">
    <restriction base="dateTime" />
  </simpleType>
```

Ref: [`dkhm-3.2.xsd`][DKHMXSD3.2]

:warning: The reference and file mentioned above is not released at this time, so this file might be re-versioned upon release.

The complete command will look as follows (example lifted from RFC:5731):

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
  <epp xmlns="urn:ietf:params:xml:ns:epp-1.0">
    <command>
      <delete>
        <domain:delete xmlns:domain="urn:ietf:params:xml:ns:domain-1.0">
          <domain:name>eksempel.dk</domain:name>
          </domain:delete>
      </delete>
      <clTRID>ABC-12345</clTRID>
    </command>
  </epp>
```

And the complete command with a deletion date specification (example lifted from RFC:5731 and modified):

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
  <epp xmlns="urn:ietf:params:xml:ns:epp-1.0">
    <command>
      <delete>
        <domain:delete xmlns:domain="urn:ietf:params:xml:ns:domain-1.0">
          <domain:name>eksempel.dk</domain:name>
          </domain:delete>
      </delete>
      <extension>
        <dkhm:delDate xmlns:dkhm="urn:dkhm:xml:ns:dkhm-3.2">2021-01-31T00:00:00.0Z</dkhm:delDate>
      </extension>
      <clTRID>ABC-12345</clTRID>
    </command>
  </epp>
```

Domain names are not deleted immediately, but are flagged as _scheduled for deletion_. This of the `delete command` is successful, the domain name will be flagged for deletion within the timeframe specified by the business rules implemented by DK Hostmaster.

The scheduling of a future delete date supports the handling of automatic renewal or expiration as outlined in "[DKHM RFC for handling of Automatic Renewal and Expiration][DKHMRFCAUTORENEW]".

The response for a `delete domain` command will be `1001`.

Response example (example lifted from RFC:5731 and modified):

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
  <epp xmlns="urn:ietf:params:xml:ns:epp-1.0">
    <response>
      <result code="1001">
        <msg>Command completed successfully; action pending</msg>
      </result>
      <trID>
        <clTRID>ABC-12345</clTRID>
        <svTRID>54321-XYZ</svTRID>
      </trID>
    </response>
  </epp>
```

The expiration date will be adjusted accordingly and a status `pendingDelete` with an advisory date will be applied and made available via the response to the `info domain` command, via the DK Hostmaster extension: `domainAdvisory`.

Example:

```xml
<extension>
    <dkhm:domainAdvisory advisory="pendingDeletionDate" date="2021-01-31T00:00:00.0Z" domain="eksempel.dk" xmlns:dkhm="urn:dkhm:params:xml:ns:dkhm-3.2"/>
</extension>
```

In the RFC outlining automatic renewal "[DKHM RFC for handling of Automatic Renewal and Expiration][DKHMRFCAUTORENEW]", it is described that a `delete domain` command will disable auto renewal if enabled. Please see the RFC for more details.

Do note that if subordinates exist these will block for a delete and the request will result in an error: `2305`.

<a id="restore"></a>
### Restore

As described in [RFC:3915][RFC3915], with a support for grace periods, it is possible to restore a domain name scheduled for deletion, (in the state `pendingDelete`).

DK Hostmaster will support the ability to restore for two use-cases:

- Get a domain name back to the state active from a pending deletion specified by an explicit deletion request (delete command) or a automatic expiration
- Get a domain name back to state active from a pending deletion, caused by missing financial settlement

Domain names might be suspended for other reasons, these will no be recoverable using the described restore facility, this will be indicated using the `serverUpdateProhibited` status.

Restoration has to take place during the redemption period and will not be possible after the grace period has expired.

The restoration is requested using the update domain command.

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<epp xmlns="urn:ietf:params:xml:ns:epp-1.0"
     xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="urn:ietf:params:xml:ns:epp-1.0 epp-1.0.xsd">
    <command>
        <update>
            <domain:update xmlns:domain="urn:ietf:params:xml:ns:domain-1.0" xsi:schemaLocation="urn:ietf:params:xml:ns:domain-1.0 domain-1.0.xsd">
                <domain:name>example.com</domain:name>
                <domain:chg/>
            </domain:update>
        </update>
        <extension>
            <rgp:update xmlns:rgp="urn:ietf:params:xml:ns:rgp-1.0" xsi:schemaLocation="urn:ietf:params:xml:ns:rgp-1.0 rgp-1.0.xsd">
                <rgp:restore op="request"/>
            </rgp:update>
        </extension>
        <clTRID>ABC-12345</clTRID>
    </command>
</epp>
```

Example is lifted from [RFC:3915][RFC:3915]

The interesting part is, the extension specifying the restore operation.

```xml
<rgp:update xmlns:rgp="urn:ietf:params:xml:ns:rgp-1.0" xsi:schemaLocation="urn:ietf:params:xml:ns:rgp-1.0 rgp-1.0.xsd">
    <rgp:restore op="request"/>
</rgp:update>
```

This will bring the domain name into the state of: `pendingRestore` for the restore, but the domain remains in: `pendingDelete`.

Next step is to acknowledge the restore operation using a report operation, which look as follows:

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<epp xmlns="urn:ietf:params:xml:ns:epp-1.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="urn:ietf:params:xml:ns:epp-1.0
     epp-1.0.xsd">
    <command>
        <update>
            <domain:update xmlns:domain="urn:ietf:params:xml:ns:domain-1.0" xsi:schemaLocation="urn:ietf:params:xml:ns:domain-1.0 domain-1.0.xsd">
                <domain:name>example.com</domain:name>
                <domain:chg/>
            </domain:update>
        </update>
        <extension>
            <rgp:update xmlns:rgp="urn:ietf:params:xml:ns:rgp-1.0" xsi:schemaLocation="urn:ietf:params:xml:ns:rgp-1.0 rgp-1.0.xsd">
                <rgp:restore op="report">
                    <rgp:report>
                        <rgp:preData></rgp:preData>
                        <rgp:postData></rgp:postData>
                        <rgp:delTime>2003-07-10T22:00:00.0Z</rgp:delTime>
                        <rgp:resTime>2003-07-20T22:00:00.0Z</rgp:resTime>
                        <rgp:resReason></rgp:resReason>
                        <rgp:statement></rgp:statement>
                    </rgp:report>
                </rgp:restore>
            </rgp:update>
        </extension>
        <clTRID>ABC-12345</clTRID>
    </command>
</epp>
```

Example is lifted from [RFC:3915][RFC:3915]

The proposal is to the the report part act as an acknowledgement. The domain name is restored _as-is_ if possible, so the mandatory fields:

- `rgp:preData`
- `rgp:postData`
- `rgp:resReason`
- `rgp:statement`

Have to be specified, but values are ignored. As are the optional field, which however is optional and does not have to be specified:

- `rgp:other`

The mandatory fields:

- `rgp:delTime`
- `rgp:resTime`

Have to be specified and will be evaluated according to [RFC:3915][RFC3915].

- The `rgp:delTime` value has to match the deletion date and time.
- The `rgp:resTime` value has to match date and time of the initial restore request (see above).

As described in [RFC:3915][RFC3915], multiple report requests can be submitted, until success and within the allowed timeframe of possible restoration.

The process described in [RFC:3915][RFC3915]. The resolution of the `rgp:resTime` might seem at bit to strict and perhaps this part of the validity check can be relaxed.

A response indicating unsuccessful restoration attempt will look as follows:

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<epp xmlns="urn:ietf:params:xml:ns:epp-1.0">
    <response>
        <result code="2004">
            <msg>Restore date is incorrect</msg>
        </result>
        <trID>
            <clTRID>ABC-12345</clTRID>
            <svTRID>54321-XYZ</svTRID>
        </trID>
    </response>
</epp>
```

Example lifted from [RFC:5730|RFC5730] and modified.

A response indicating successful restoration attempt will look as follows:

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<epp xmlns="urn:ietf:params:xml:ns:epp-1.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="urn:ietf:params:xml:ns:epp-1.0
     epp-1.0.xsd">
    <response>
        <result code="1000">
            <msg lang="en">Command completed successfully</msg>
        </result>
        <extension>
            <rgp:upData xmlns:rgp="urn:ietf:params:xml:ns:rgp-1.0" xsi:schemaLocation="urn:ietf:params:xml:ns:rgp-1.0 rgp-1.0.xsd">
                <rgp:rgpStatus s="pendingRestore"/>
            </rgp:upData>
        </extension>
        <trID>
            <clTRID>ABC-12345</clTRID>
            <svTRID>54321-XYZ</svTRID>
        </trID>
    </response>
</epp>
```

Example is lifted from [RFC:3915][RFC:3915]

<a id="xsd-definition"></a>
## XSD Definition

This XSD definition is for the proposed extension `dkhm:delDate`, which is used to communicate a deletion date differing from the default via the `delete domain` request.

```xsd
  <!-- custom: delDate  -->
  <simpleType name="delDate">
    <restriction base="dateTime" />
  </simpleType>
```

Example (lifted from above):

```xml
  <extension>
    <dkhm:delDate xmlns:dkhm="urn:dkhm:xml:ns:dkhm-3.2">2021-01-31T00:00:00.0Z</dkhm:delDate>
  </extension>
```

Ref: [`dkhm-3.2.xsd`][DKHMXSD3.2]

:warning: The reference and file mentioned above is not released at this time, so this file might be re-versioned upon release.

<a id="references"></a>
## References

- [DK Hostmaster EPP Service Specification][DKHMEPPSPEC]
- [DK Hostmaster EPP Service XSD Repository][DKHMXSDSPEC]
- [RFC:3339: "Date and Time on the Internet: Timestamps"][RFC3339]
- [RFC:5730 "Extensible Provisioning Protocol (EPP)"][RFC5730]
- [RFC:5731 "Extensible Provisioning Protocol (EPP) Domain Name Mapping"][RFC5731]

[RFC5730]: https://www.rfc-editor.org/rfc/rfc5730.html
[RFC5731]: https://www.rfc-editor.org/rfc/rfc5731.html
[RFC3339]: https://www.rfc-editor.org/rfc/rfc3339.html
[RFC3915]: https://tools.ietf.org/html/rfc3915.html
[RFC8748]: https://tools.ietf.org/html/rfc8748.html
[DKHMEPPSPEC]: https://github.com/DK-Hostmaster/epp-service-specification
[DKHMXSDSPEC]: https://github.com/DK-Hostmaster/epp-xsd-files
[DKHMRFCAUTORENEW]: https://github.com/DK-Hostmaster/DKHM-RFC-AutoRenew
[CONCEPT]: https://www.dk-hostmaster.dk/en/new-basis-collaboration-between-registrars-and-dk-hostmaster
[DKHMXSD3.2]: https://github.com/DK-Hostmaster/epp-xsd-files/blob/master/dkhm-3.2.xsd
