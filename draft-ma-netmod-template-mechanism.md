---
title: "YANG Templates"
abbrev: "template"
category: std

docname: draft-ma-netmod-template-mechanism-latest
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

normative:

informative:


--- abstract

TODO Abstract


--- middle

# Introduction

TODO Introduction


# Conventions and Definitions

{::boilerplate bcp14-tagged}

# The "ietf-template" YANG Module

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

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
