![DK Hostmaster Logo](https://www.dk-hostmaster.dk/sites/default/files/dk-logo_0.png)

# DKHM RFC for Delete Domain EPP Command

![GitHub Workflow build status badge markdownlint](https://github.com/DK-Hostmaster/DKHM-RFC-Delete-Domain/workflows/Markdownlint%20Workflow/badge.svg)

## Table of Contents

<!-- MarkdownTOC bracket=round levels="1,2,3,4" indent="  " autolink="true" autoanchor="true" -->

- [Introduction](#introduction)
  - [About this Document](#about-this-document)
  - [XML and XSD Examples](#xml-and-xsd-examples)
- [Description](#description)
  - [Setting and Unsetting the AuthInfo Token](#setting-and-unsetting-the-authinfo-token)
  - [Fetching the AuthInfo via EPP](#fetching-the-authinfo-via-epp)
  - [Name Server Change Request](#name-server-change-request)
  - [Requesting a Name Server Change Using AuthInfo via a Pull Operation](#requesting-a-name-server-change-using-authinfo-via-a-pull-operation)
- [XSD Definition](#xsd-definition)
- [Scenario Matrix](#scenario-matrix)
  - [AuthInfo Token Access](#authinfo-token-access)
  - [Generation of AuthInfo token with Registry](#generation-of-authinfo-token-with-registry)
  - [Change of Name Server, pull \(external and internal\)](#change-of-name-server-pull-external-and-internal)
- [AuthInfo Token Format](#authinfo-token-format)
- [References](#references)

<!-- /MarkdownTOC -->

<a id="introduction"></a>
## Introduction

This is a draft and proposal for changes to the proces for domain name deletion via the DK Hostmaster EPP portal/service.

<a id="about-this-document"></a>
### About this Document

We have adopted the term RFC (_Request For Comments_), due to the recognition in the term and concept, so this document is a process supporting document, aiming to serve the purpose of obtaining a common understanding of the proposed implementation and to foster discussion on the details of the implementation. The final specification will be lifted into the [DK Hostmaster EPP Service Specification](https://github.com/DK-Hostmaster/epp-service-specification) implementation and this document will be closed for comments and the document no longer be updated.

<a id="xml-and-xsd-examples"></a>
### XML and XSD Examples

All example XML files are available in the [DK Hostmaster EPP XSD repository](https://github.com/DK-Hostmaster/epp-xsd-files) in the [delete-domain-dkhm-extension](https://github.com/DK-Hostmaster/epp-xsd-files/tree/delete-domain-dkhm-extension).

<a id="description"></a>
## Description

```xsd
  <!-- custom: delDate  -->
  <simpleType name="delDate">
    <restriction base="dateTime" />
  </simpleType>
```

Example (lifted from above):

```xml
    <extension>
      <dkhm:delDate xmlns:dkhm="urn:dkhm:params:xml:ns:dkhm-3.2">2020-01-31T00:00:00.0Z</dkhm:delDate>
    </extension>
```

Ref: [`dkhm-3.2.xsd`](https://raw.githubusercontent.com/DK-Hostmaster/epp-xsd-files/master/dkhm-3.2.xsd)

<a id="references"></a>
## References

- [DK Hostmaster EPP Service Specification](https://github.com/DK-Hostmaster/epp-service-specification)
- [DK Hostmaster EPP Service XSD Repository](https://github.com/DK-Hostmaster/epp-xsd-files)