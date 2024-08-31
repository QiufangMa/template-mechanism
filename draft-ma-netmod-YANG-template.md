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
By defining a common set of nodes as a configuration template and applying the
configuration template repeatedly, it avoids the redundant definition of identical
configuration and also ensures consistency of it. Configuration template could
be used based on any existing YANG data models, this document doesn't make any
assumption on the YANG data model design, i.e., does not rely on the shared profile/group
defined in the YANG data model.

## Editorial Note (To be removed by RFC Editor)

Note to the RFC Editor: This section is to be removed prior to publication.

This document contains placeholder values that need to be replaced with finalized
values at the time of publication.  This note summarizes all of the
substitutions that are needed.  No other RFC Editor instructions are specified
elsewhere in this document.

Please apply the following replacements:

   * XXXX --> the assigned RFC number for this draft
   * 2024-08-27 --> the actual date of the publication of this document

# Conventions and Definitions

{::boilerplate bcp14-tagged}

The meanings of the symbols in tree diagrams are defined in
{{?RFC8340}}.

This document uses the YANG terminology defined in {{Section 3 of !RFC7950}}.

Besides, this document defines the following terminology:

configuration template:
: A chunk of reusable configuration data that could be applied to the configuration
  repeatedly, in order to simplify the delivery of network configuration and
  ensure the consistency of it. Configuration templates can be applied at different
  levels of the data tree, a configuration template can also be called
  "hierarchical template", or "template" for short.

inherited template:
: A configuration template that is applied in the configuration of network
  devices or reused by other templates.

parent template:
: A configuration template that is an inherited template.

# Hierarchical Template Overview

> Editor's Note: The RPC may not be needed as operator can refer to the expansion by querying \<intended\>.

A configuration template must first be defined before it can be applied. The creation,
modification, and deletion of configuration templates can be achieved by network
management operations via NETCONF or RESTCONF protocols. The content of the configuration
template must be an instantiated chunk of data. For example, {{temp-ex}} provides
an interface configuration template that MTU is set as 1500 for Ethernet interfaces:

~~~~
<type>ianaift:ethernetCsmacd<type>
<mtu>1500<mtu>
~~~~
{: #temp-ex title="Example of An Interface template 'eth-mtu'"}


The YANG data model of configuration templates is defined in {{template-yang}}.

## Validity of Templates

The contents of the template alone do not always follow the constraints
of the data model. Some constraints may depend on configuration outside of the
templates to satisfy, e.g., a list may contain a mandatory leaf node which is not
defined in the template but explicitly provided by the client. However, servers
should parse the template and enforce the constraints if possible during the
processing of template creation, e.g., type constraints for the leaf,
including those defined in the type's "range", "length", and "pattern" properties.

That said, if a template is applied in the data tree, the results of the template
configuration merging with configuration explicitly provided by the client MUST
always be a valid configuration data tree, as defined in {{Section 8.1 of !RFC7950}}.

## Different Levels of Templates

The configuration templates may be at different levels, depending on
where it is used and maintained.
For exmaple, A network-level template maintained by the software-defined networking
(SDN) {{?RFC7149}} {{?RFC7426}} controller defines configuration that may be shared
by multiple network devices. While device-level template maintained by the network
element defines configuration that can only be applied to specific network devices.
Refer to {{appendix-network}} for examples of network-level templates.

# Inheriting Templates

This document allows configuration templates to be inherited by
configuration nodes in the data tree or new templates.

If a configuration template is inherited by a node in the data tree, it acts as
if the configuration defined in the template is contained as the child configuration
of that node and merged with the configuration at the corresponding level in the data tree.

If a configuration template is inherited by another new template,
the configuration of the new template is the merging result of configuration defined
in both templates with the new template takes precedence over its parent template.
This is useful when some additional configuration is intended to be defined on the
basis of the parent template.

Any modification to the parent template also applies somewhere inherits the template.
Care MUST be taken when making changes to the parent templates.

## The "stmt-extend" Metadata

Template inheritance is flagged by declaring the metadata object called "stmt-extend".
If the template is inherited by a node in the data tree, the metadata object is added
to that specific node. A instance node inherits at most one configuration template.

If the template is inherited by another template, the metadata
object is added to the "/ietf-template:templates/ietf-template:template" instance node that specifies the template contents.
Similarly, a template inherits at most one configuration template.

The "stmt-extend" metadata MUST have only one value to specify the parent template
identifier that is inherited. The encoding of "stmt-extend" metadata object follows the way defined
in {{Section 5 of ?RFC7952}}.

For example, a client may configure a physically present interface "eth0"
by inheriting the template defined in {{temp-ex}}:

~~~~
<interfaces xmlns:template="urn:ietf:params:xml:ns:yang:ietf-template">  
  <interface template:stmt-extend="eth-mtu">
    <name>eth0</name>
  </interface>
</interfaces>
~~~~

{{template-inherits}} provides more examples of inheriting an existing template by indicating
the "stmt-extend" metadata object.

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

## Interaction with NMDA datastores

Some implementation may have predefined configuration templates for the convenience
of clients, which are present in \<system\> (if implemented, see {{?I-D.ietf-netmod-system-config}}).
In addition, clients can always define their own templates in \<running\>.
However, configuration template data defined by "ietf-template" YANG data model
should not be visible in \<operational\> until being inherited by a node in the data tree.

If a node in the data tree inherits a configuration template, the configuration
template does not expand in \<running\>, a read back of \<running\> returns what is
sent by the client with the "stmt-extend" metadata attached to the specific node.
Configuration template which is inherited or overridden by the node instance MUST be expanded in \<intended\>.

> Editor's Note: The read-back of \<running\> might break legacy clients doesn't
understand template?


## The "get-template-expansion" RPC

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

# Usage Examples {#appendix-network}

This section provides some examples to show the use of templates.
Both NETCONF and RESTCONF protocol operations and different encodings are used
to not imply a preference in this document.
The fictional data model used throughout this section is as follows. This
network model is inspired by the device model defined in {{?RFC7317}}.

~~~~
{::include-fold ./yang/example-network-systime.yang}
~~~~

## Creating Templates {#template-creation}

The NTP configuration on multiple network devices may be consistent. To create a
configuration template, the following RESTCONF request might be sent to a SDN controller:

~~~~
POST /restconf/data HTTP/1.1
HOST: example.com
Content-Type: application/yang-data+json

{
    "ietf-templates:templates": {
        "template": [
            {
                "id": "template-ntp",
                "content": {
                    "ntp": {
                        "enabled": "true",
                        "server": [
                            {
                                "name": "ntp-service-primary",
                                "alias": "primary",
                                "address": "ntp.service.com",
                                "prefer": true
                            },
                            {
                                "name": "ntp-service-secondary",
                                "alias": "secondary",
                                "address": "ntp.service.backup.com"
                            }
                        ]
                    }
                }
            }
        ]
    }
}
~~~~

## Inheriting An Existing Template {#template-inherits}

The operator may create another template with an additional NTP server instance
by inheriting the template created in {{template-creation}}. Only the message-body
is shown as follows:

~~~~
{
    "ietf-templates:templates": {
        "template": [
            {
                "@": {
                    "ietf-template:stmt-extend": "template-ntp"
                },
                "id": "template-ntp2",
                "content": {
                    "ntp": {
                        "server": [
                            {
                                "name": "ntp-service-secondary-2",
                                "alias": "secondary",
                                "address": "ntp.service.backup.com"
                            }
                        ]
                    }
                }
            }
        ]
    }
}
~~~~

The configuration of template "template-ntp2" is equivalent to the following:

~~~~
{
    "ntp": {
        "enabled": "true",
        "server": [
            {
                "name": "ntp-service-primary",
                "alias": "primary",
                "address": "ntp.service.com",
                "prefer": true
            },
            {
                "name": "ntp-service-secondary",
                "alias": "secondary",
                "address": "ntp.service.backup.com"
            },
            {
                "name": "ntp-service-secondary-2",
                "alias": "secondary",
                "address": "ntp.service.backup.com"
            }
        ]
    }
}
~~~~

the following shows the network-level ntp configuration with JSON format
using "template-ntp" and "template-ntp2" that may be sent to a SDN controller:

~~~~
{
    "example-network-systime:network-device": [
        {
            "@": {
                "ietf-template:stmt-extend": "template-ntp"
            },
            "device-id": "ne-0"
        },
        {
            "@": {
                "ietf-template:stmt-extend": "template-ntp"
            },
            "device-id": "ne-1"
        },
        {
            "@": {
                "ietf-template:stmt-extend": "template-ntp2"
            },
            "device-id": "ne-2"
        },
        {
            "@": {
                "ietf-template:stmt-extend": "template-ntp2"
            },
            "device-id": "ne-3"
        }
    ]
}
~~~~

Which is equivalent to the following:

~~~~
{
    "example-network-systime:network-device": [
        {
            "device-id": "ne-0",
            "ntp": {
                "enabled": "true",
                "server": [
                    {
                        "name": "ntp-service-primary",
                        "alias": "primary",
                        "address": "ntp.service.com",
                        "prefer": true
                    },
                    {
                        "name": "ntp-service-secondary",
                        "alias": "secondary",
                        "address": "ntp.service.backup.com"
                    }
                ]
            }
        },
        {
            "device-id": "ne-1",
            "ntp": {
                "enabled": "true",
                "server": [
                    {
                        "name": "ntp-service-primary",
                        "alias": "primary",
                        "address": "ntp.service.com",
                        "prefer": true
                    },
                    {
                        "name": "ntp-service-secondary",
                        "alias": "secondary",
                        "address": "ntp.service.backup.com"
                    }
                ]
            }
        },
        {
            "device-id": "ne-2",
            "ntp": {
                "enabled": "true",
                "server": [
                    {
                        "name": "ntp-service-primary",
                        "alias": "primary",
                        "address": "ntp.service.com",
                        "prefer": true
                    },
                    {
                        "name": "ntp-service-secondary",
                        "alias": "secondary",
                        "address": "ntp.service.backup.com"
                    },
                    {
                        "name": "ntp-service-secondary-2",
                        "alias": "secondary",
                        "address": "ntp.service.backup.com"
                    }
                ]
            }
        },
        {
            "device-id": "ne-3",
            "ntp": {
                "enabled": "true",
                "server": [
                    {
                        "name": "ntp-service-primary",
                        "alias": "primary",
                        "address": "ntp.service.com",
                        "prefer": true
                    },
                    {
                        "name": "ntp-service-secondary",
                        "alias": "secondary",
                        "address": "ntp.service.backup.com"
                    },
                    {
                        "name": "ntp-service-secondary-2",
                        "alias": "secondary",
                        "address": "ntp.service.backup.com"
                    }
                ]
            }
        }
    ]
}
~~~~

## Overriding An Existing Template

## Expanding Templates

An operation request to expand the "template-ntp2" template in {{template-inherits}}:

~~~~
<rpc message-id="101"
     xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
  <get-template-expansion>
    <template-id>template-ntp2</template-id>
  </get-template-expansion>
</rpc>

<rpc-reply message-id="101"
     xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
  <data>
    <ntp>
      <enabled>true</enabled>
      <server>
        <name>ntp-service-primary</name>
        <alias>primary</alias>
        <address>ntp.service.com</address>
        <prefer>true</prefer>
      </server>
      <server>
        <name>ntp-service-secondary</name>
        <alias>secondary</alias>
        <address>ntp.service.backup.com</address>
      </server>
      <server>
        <name>ntp-service-secondary-2</name>
        <alias>secondary</alias>
        <address>ntp.service.backup.com</address>
      </server>
    </ntp>
  </data>
</rpc-reply>
~~~~

## Applying Templates

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
