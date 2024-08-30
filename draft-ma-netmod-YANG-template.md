---
title: "YANG Templates"
abbrev: "template"
category: std

docname: draft-ma-netmod-YANG-template-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Operations and Management"
workgroup: "Network Modeling"
keyword:
 - YANG
 - template
 - NMDA

author:
-
   fullname: Qiufang Ma
   organization: Huawei
   street: 101 Software Avenue, Yuhua District
   city: Nanjing, Jiangsu
   code: 210012
   country: China
   email: maqiufang1@huawei.com
-
  fullname: Zhan Wu
  organization: Huawei
  email: wuzhan1@huawei.com

normative:

informative:


--- abstract

NETCONF and RESTCONF protocols provide programmatic operation interfaces for accessing
configuration data modeled by YANG. This document defines the use of YANG-based
configuration template so that the configuration data could be delivered in a more
efficient manner.

--- middle

# Introduction

It is not unusual for the YANG data model {{!RFC7950}} to define some shared profiles that could
be referenced in order to simplify the configuration of network services or functionalities.
For example, {{?I-D.ietf-opsawg-ntw-attachment-circuit}} defines a set of profiles
at the network level which could be referred to under the node level to factorize
some common configuration shared by a group of ACs.

However, it is not trivial to always take care of the definition of shared profiles/
policies/templates in the design of every data model that could benefit from them.
There is a desire to make use of common YANG-based templates without relying on
specific definition in YANG data models.

NMDA {{?RFC8342}} allows the configuration templates to be defined in \<running\>
and expanded in \<intended\>, but it does not specify details about how configuration
templates could be created and reused.

This document defines the use of configuration templates in the context of YANG-driven
network management protocols such as NETCONF {{!RFC6241}} and RESTCONF {{!RFC8040}}.
Template data could be created based on any existing YANG data models.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

The meanings of the symbols in tree diagrams are defined in
{{?RFC8340}}.

This document uses the YANG terminology defined in {{Section 3 of !RFC7950}}.

Besides, this document defines the following terminology:

configuration template:
: A chunk of reusable configuration data that could be applied to the configuration
  of network devices repeatedly, in order to simplify the delivery of network configuration and
  ensure the consistency of it. A configuration template can also be called
  "template" for short.

inherited template:
: A configuration template that is applied in the configuration of network
  devices or other templates.

parent template:
: A configuration template that is inherited by configuration or other templates.

# Hierarchical Template Overview

> Editor's Note: Why we'd call Hierarchical Template?
> Editor's Note: The RPC may not be needed as operator can refer to the expansion by querying \<intended\>.

The configuration template may be created, modified, and deleted by management operations
via NETCONF and RESTCONF protocols. By defining additional metadata, this document
allows configuration templates to be inherited, composed, and overridden by
configuration explicitly provided by the client or new templates. In addition,
this document also defines a RPC called "get-template-expansion" to allow the
client to retrieve the expansion result of a specific configuration template.

The YANG data model is defined in {{template-yang}}.

## Validity of Templates

The contents of the template alone do not always follow the constraints (as per {{Section 8.1 of !RFC7950}})
of the data model. Some constraints may depend on configuration outside of the
templates to satisfy, e.g., a list may contain a mandatory leaf node which is not
defined in the template but explicitly provided by the client. However, servers
should parse the template and enforce the constraints if possible during the
processing of template creation, e.g., type constraints for the leaf,
including those defined in the type's "range", "length", and "pattern" properties.

The results of inherited template merging with configuration explicitly provided by the client MUST
always be a valid configuration data tree.

## Network-level Templates VS. Device-level Templates

> Editor's Note: This section may not be needed.

The configuration templates may be network-level or device-level, depending on
whether it is maintained at the network controller or network elements.
A network-level template can be shared across multiple network devices.
Device-level template can only be applied to specific network devices.

# Creating Templates

Templates are defined as needed, for example, when it needs
to deliver some identical configuration to multiple network elements.

{{template-creation}} provides an example of template creation.

## Variables

In some cases, there are some customized configuration that varies.

# Inheriting Templates

A template is inherited if it is intended to be applied in the configuration of
the device or other templates.
If a configuration template is inherited, it is called a "parent template".
Any modification to it also applies somewhere inherits the template.
Care MUST be taken when making changes to the parent templates.

## The "stmt-extend" Metadata

# Composing templates

## The "stmt-compose" Metadata

# Overriding Templates

## Creation

## Modification

## Deletion

## List/leaflists Reordering

### Position_first

### Position_last

### Position_before

### Position_after

## Expanding Templates

### The "get-template-expansion" RPC

## Applying Templates




# The "ietf-template" YANG Module {#template-yang}

## Data Model Overview

The following tree diagram {{?RFC8340}} illustrates the "ietf-template" module:

~~~~
{::include ./yang/ietf-template-tree.txt}
~~~~

## YANG Module

~~~~
<CODE BEGINS> file "ietf-template@2024-08-27.yang"
{::include-fold ./yang/ietf-template.yang}
<CODE ENDS>
~~~~

# Security Considerations

TODO Security


# IANA Considerations

##  The "IETF XML" Registry

   This document registers the following URI in the "IETF XML Registry" {{!RFC3688}}.

~~~~
        URI: urn:ietf:params:xml:ns:yang:ietf-template
        Registrant Contact: The IESG.
        XML: N/A, the requested URI is an XML namespace.
~~~~

##  The "YANG Module Names" Registry

   This document registers the following YANG module in the "YANG Module Names"
   registry {{!RFC6020}}.

~~~~
        name:               ietf-template
        namespace:          urn:ietf:params:xml:ns:yang:ietf-template
        prefix:             template
        maintained by IANA? N
        reference:          RFC XXXX
~~~~


--- back

# Usage Examples

This section provides some examples to show the use of templates.
JSON encoding is used only for illustration purposes.
The fictional data model used throughout this section is as follows. This
network model is inspired by the device model defined in {{?RFC7317}}.

~~~~
{::include-fold ./yang/example-network-systime.yang}
~~~~

## Creating Templates {#template-creation}

The NTP configuration on multiple network devices may be consistent.

## Inheriting An Existing Template

## Overriding An Existing Template

## Expanding Templates

## Applying Templates

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
