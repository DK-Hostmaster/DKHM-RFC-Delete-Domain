![DK Hostmaster Logo](https://www.dk-hostmaster.dk/sites/default/files/dk-logo_0.png)

# DKHM RFC for Delete Domain EPP Command

![Markdownlint Action](https://github.com/DK-Hostmaster/DKHM-RFC-Delete-Domain/workflows/Markdownlint%20Action/badge.svg)
![Spellcheck Action](https://github.com/DK-Hostmaster/DKHM-RFC-Delete-Domain/workflows/Spellcheck%20Action/badge.svg)

2020-11-23
Revision: 1.4

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

This document is not the authoritative source for business and policy rules and possible discrepancies between this an any authoritative sources are regarded as errors in this document. This document is aimed at the technical specification and possible implementation and is an interpretation of authoritative sources and can therefor be erroneous.

<a id="license"></a>
### License

This document is copyright by DK Hostmaster A/S and is licensed under the MIT License, please see the separate LICENSE file for details.

<a id="document-history"></a>
### Document History

- 1.4 2020-11-23
  - Removal of restoration information, which has been collected in a separate RFC
  - Addition of additional links to resources
  - Correction to links pointing to redundant resources
  - Minor rephrasing and clarifications

- 1.3 2020-09-17
  - Addition of disclaimer

- 1.2 2020-09-17
  - Added information on restore as described in [RFC:3915] and [RFC:8748]

- 1.1 2020-09-01
  - Clarification on deletion and handling of subordinates, as specified in [RFC:5731]

- 1.0 2020-08-25
  - Initial revision

<a id="xml-and-xsd-examples"></a>
### XML and XSD Examples

All example XML files are available in the [DK Hostmaster EPP XSD repository][DKHMXSDSPEC].

The proposed extensions and XSD definitions are available in version [4.0][DKHMXSD4.0] of the DK Hostmaster XSD, which is marked as a  _pre-release_.

The referenced XSD version is not deployed at this time and is only available in the [EPP XSD repository][DKHMXSDSPEC], it might be surpassed by a newer version upon deployment of the EPP service implementing the proposal, please refer to the revision of [EPP Service Specification][DKHMEPPSPEC] describing the implementation.

<a id="description"></a>
## Description

<a id="delete-and-scheduling-deletion"></a>
### Delete and Scheduling Deletion

In addition to the standard EPP `delete domain` command, DK Hostmaster will support scheduling of deletion of domain names, by providing a date to the EPP `delete domain` command via an optional extension.

The default is to deactivate immediately if possible, which complies with [RFC:5731]. Not being able to complete the request will result in a error, also in compliance with [RFC:5731]. Please see below for more information on the business process for deletion.

The extension offers the ability to specify a date, this date will have to be in the future and prior to, or on the expiration date of the specified domain name.

The current expiration date can be obtained using the `info domain` command and is specified in the `domain:exDate` field. The date conforms with the required format.

An example (do note the dates in the below examples are examples and are fabricated and might not be correct, meaning they might be in the past by the time of reading):

```xml
  <extension>
    <dkhm:delDate xmlns:dkhm="urn:dkhm:xml:ns:dkhm-3.2">2021-01-31T00:00:00.0Z</dkhm:delDate>
  </extension>
```

The date follows the format used in the EPP protocol, which complies with [RFC:3339].

The XSD for the extension look as follows:

```xsd
  <!-- custom: delDate  -->
  <simpleType name="delDate">
    <restriction base="dateTime" />
  </simpleType>
```

Ref: [`dkhm-4.0.xsd`][DKHMXSD4.0]

:warning: The reference and file mentioned above is not released at this time, so this file might be re-versioned upon release.

The complete command will look as follows (example lifted from [RFC:5731]):

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

And the complete command with a deletion date specification (example lifted from [RFC:5731] and modified):

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
        <dkhm:delDate xmlns:dkhm="urn:dkhm:xml:ns:dkhm-4.0">2021-01-31T00:00:00.0Z</dkhm:delDate>
      </extension>
      <clTRID>ABC-12345</clTRID>
    </command>
  </epp>
```

Domain names are not deleted immediately, but are flagged as _scheduled for deletion_. This of the `delete command` is successful, the domain name will be flagged for deletion within the timeframe specified by the business rules implemented by DK Hostmaster.

The scheduling of a future delete date supports the handling of automatic renewal or expiration as outlined in "[DKHM RFC for handling of Automatic Renewal and Expiration][DKHMRFCAUTORENEW]".

The response for a `delete domain` command will be `1001`.

Response example (example lifted from [RFC:5731] and modified):

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
    <dkhm:domainAdvisory advisory="pendingDeletionDate" date="2021-01-31T00:00:00.0Z" domain="eksempel.dk" xmlns:dkhm="urn:dkhm:params:xml:ns:dkhm-4.0"/>
</extension>
```

In the RFC outlining automatic renewal "[DKHM RFC for handling of Automatic Renewal and Expiration][DKHMRFCAUTORENEW]", it is described that a `delete domain` command will disable auto renewal if enabled. Please see the RFC for more details.

Do note that if subordinates exist these will block for a delete and the request will result in an error: `2305`.

<a id="restore"></a>
### Restore

As described in [RFC:3915], with a support for grace periods, it is possible to restore a domain name scheduled for deletion, (in the state `pendingDelete`).

The ability to restore a domain from a suspended state, is mentioned in the introduction as a billable operation.

The proposal for the restore implementation is outlined in the separate RFC: ["DKHM RFC for Restore Command"][DKHMRFCRESTORE].

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
    <dkhm:delDate xmlns:dkhm="urn:dkhm:xml:ns:dkhm-4.0">2021-01-31T00:00:00.0Z</dkhm:delDate>
  </extension>
```

Ref: [`dkhm-4.0.xsd`][DKHMXSD4.0]

The referenced XSD version is not deployed at this time and is only available in the [EPP XSD repository][DKHMXSDSPEC], it might be surpassed by a newer version upon deployment of the EPP service implementing the proposal, please refer to the revision of [EPP Service Specification][DKHMEPPSPEC] describing the implementation.

<a id="references"></a>
## References

1. ["New basis for collaboration between registrars and DK Hostmaster"][CONCEPT]
1. [DK Hostmaster EPP Service Specification][DKHMEPPSPEC]
1. [DK Hostmaster EPP Service XSD Repository][DKHMXSDSPEC]
1. [DKHM RFC for handling of Automatic Renewal or Expiration][DKHMRFCAUTORENEW]
1. [DKHM RFC for Restore Command][DKHMRFCRESTORE]
1. [RFC:3339: "Date and Time on the Internet: Timestamps"][RFC:3339]
1. [RFC:3915 "Domain Registry Grace Period Mapping for the Extensible Provisioning Protocol (EPP)"][RFC:3915]
1. [RFC:5730 "Extensible Provisioning Protocol (EPP)"][RFC:5730]
1. [RFC:5731 "Extensible Provisioning Protocol (EPP) Domain Name Mapping"][RFC:5731]
1. [RFC:8748 "Registry Fee Extension for the Extensible Provisioning Protocol (EPP)"][RFC:8748]

[CONCEPT]: https://www.dk-hostmaster.dk/en/new-basis-collaboration-between-registrars-and-dk-hostmaster
[DKHMEPPSPEC]: https://github.com/DK-Hostmaster/epp-service-specification
[DKHMXSDSPEC]: https://github.com/DK-Hostmaster/epp-xsd-files
[DKHMRFCAUTORENEW]: https://github.com/DK-Hostmaster/DKHM-RFC-AutoRenew
[DKHMRFCRESTORE]: https://github.com/DK-Hostmaster/DKHM-RFC-Restore
[RFC:3339]: https://tools.ietf.org/html/rfc3339
[RFC:3915]: https://tools.ietf.org/html/rfc3915
[RFC:5730]: http://tools.ietf.org/html/rfc5730
[RFC:5731]: http://tools.ietf.org/html/rfc5731
[RFC:8748]: https://tools.ietf.org/html/rfc8748
[DKHMXSD4.0]: https://github.com/DK-Hostmaster/epp-xsd-files/blob/master/dkhm-4.0.xsd
