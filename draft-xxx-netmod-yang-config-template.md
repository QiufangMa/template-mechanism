---
title: "YANG Templates"
abbrev: "template"
category: std

docname: draft-xxx-netmod-yang-config-template-latest
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
   role: editor
   street: 101 Software Avenue, Yuhua District
   city: Jiangsu
   code: 210012
   country: China
   email: maqiufang1@huawei.com

-
   fullname: Robert Wills
   organization: Cisco
   role: editor
   country: United Kingdom
   email: rowills@cisco.com

-
   fullname: Deepak Rajaram
   organization: Nokia
   role: editor
   country: India
   email: deepak.rajaram@nokia.com

contributor:
-
   fullname: Qin Wu
   organization: Huawei
   street: 101 Software Avenue, Yuhua District
   city: Jiangsu
   code: 210012
   country: China
   email: bill.wu@huawei.com


normative:

informative:


--- abstract

NETCONF and RESTCONF protocols provide programmatic operation interfaces for accessing
configuration data modeled by YANG. This document defines the use of YANG-based
configuration template mechanism so that the configuration data could be defined as template
and applied repeatedly to avoid the redundant definition of identical Configuration
and ensure consistency of it. This approach is both convenient and efficient,
as it minimizes the size of the running datastore and reduces network provisioning time.

--- middle

# Introduction

This document considers the case of a device that contains a functional entity, characterized
by a well-defined data nodes pattern, that is massively replicated and where each replication
instance needs individual configuration with only limited variation.Having a device manager
that repetitively configures each data node for every functional instance can become complex
and prone to errors. This approach may lead to issues, such as extended configuration times,
increased memory usage on the device, and inefficient YANG validation processes due to the
large size of the running data store. These challenges only intensify as the system scales.
This document proposes a technique to improve this, which is based on 'YANG templates'
that results in a smaller running data store even when the device is very large.

A 'YANG template' is the configuration of a functional entity that the device is instructed
to replicate multiple times to generate copies of the entity. The technique that is outlined
in this document allows to generate copies with the same data node values as in the template
with the possibility, though, to overrule some of these values on an individual copy basis.

This document describes a mechanism whereby nodes of configuration data can be placed into templates,
and templates can be applied to subtrees in a configuration datastore.
When a template is applied to a subtree, the configuration in the template takes effect for that subtree
(unless other configuration takes precedence, as described later in this document)

NMDA {{?RFC8342}} allows the configuration templates to be defined in \<running\>
and expanded in \<intended\>, but it does not specify details about how configuration
templates could be created and applied.

This document defines the use of configuration templates in the context of YANG-driven
network management protocols such as NETCONF {{!RFC6241}} and RESTCONF {{!RFC8040}}.
Configuration template could be used based on any existing YANG data models,
this document doesn't make any assumption on the YANG data model design,
i.e., it does not rely on the shared profile/group defined in the YANG data model.


## Editorial Note (To be removed by RFC Editor)

Note to the RFC Editor: This section is to be removed prior to publication.

This document contains placeholder values that need to be replaced with finalized
values at the time of publication.  This note summarizes all of the
substitutions that are needed.  No other RFC Editor instructions are specified
elsewhere in this document.

Please apply the following replacements:

   * XXXX --> the assigned RFC number for this draft
   * 2025-03-28 --> the actual date of the publication of this document

# Conventions and Definitions

{::boilerplate bcp14-tagged}

The meanings of the symbols in tree diagrams are defined in
{{?RFC8340}}.

This document uses the YANG terminology defined in {{Section 3 of !RFC7950}}.

Besides, this document defines the following terminology:

configuration template:
: A chunk of reusable configuration data that could be applied to the configuration
  repeatedly, in order to simplify the delivery of network configuration and
  ensure the consistency of it. A configuration template can also be called
  "template" for short.

inherited template:
: A configuration template that is applied in the configuration data tree.

parent template:
: A configuration template that is an inherited template.

# Requirements {#requirements}

This section describes the requirements that the Yang Templates solution must
satisfy. These requirements were all discussed in the Interim Meetings, and a
rough consensus was reached on each of them by the participants in the meetings.
A general theme of the Yang Templates work is to come up with a "Minimal Viable
Product" that is useful but not over-complicated. More advanced features could be
considered as extensions in later drafts.

## Defining and Managing Templates

Templates can be used with any Yang module.  They contain nodes of
configuration data, and are stored persistently in the running
datastore of the device.

A client can view and manipulate a template, including the
configuration inside it, by manipulating it in the \<running\>
datastore.  In this sense, a template and its contents behaves like
any other subtree of configuration.

## Applying Templates

A template can be applied to zero or more nodes in the \<running\>
datastore.  Each node can have zero or more templates applied to it,
and the order they are applied is specified by the client.  The order
is important when determining the final intended configuration -- see
the next section.

Templates can be applied at multiple points in the hierachy.  The
next section states the requirements when a node applies a template
and it has an ancestor that also applies a template.

When viewing the \<running\> datastore, there is a mechanism to see
which templates have been applied to each node, and in which order.

## Producing the Intended Datastore

The device's \<intended\> datastore is the result of combining all the
applications of templates together with non-template config.  This is
called "expanding out" the templates.

The intended configuration inside a subtree is the result of taking
the relevant contents of every template applied to the subtree's root
node and its ancestors, and combining it with the (non-template) data
nodes inside the subtree.

A node inside a subtree may be present in multiple templates that
have been applied, and/or it may be present as non-template config
inside the subtree.  The requirements for combining the templates and
the non-template config together are as follows:

*  The value of a node in the \<intended\> configuration is determined
   by using precedence to decide where to take the value from.

*  Non-template config always has the highest precedence.

*  When templates are applied to multiple ancestors, the innermost
   ancestor takes precedence.

*  When multiple templates are applied to a particular node, the
   order of application (as indicated by the client when applying the
   templates) determines the precedence within that node.

Whenever the contents of a template is updated in \<running\>, the
result of expanding out the template appears in \<intended\> and takes
effect on the device.

## Pattern Matching in Templates

The configuration inside a template definition can contain values for
list keys that are simple regular expressions, using a limited subset
of regular expression syntax.  This controls which list entries that
subtree of the template takes effect for when it is applied.

An example of this would be to have a template that is applied to a
top-level "interfaces" container, but the template only takes effect
for certain interface names that match the regular expression.

## Off-box Template Expansion

If the client knows the contents of the \<running\> datastore (non-
template config, template definitions and template applications), it
must be possible for the client to calculate the result of template
expansion.

In other words, the outcome of template expansion depends solely on
the \<running\> datastore and not the state of the device.

# YANG Template Solution

## Defining Templates

A configuration template must first be defined before it can be inherited {{inheriting-temp}}. The creation,
modification, and deletion of configuration templates are achieved by network
management operations via NETCONF or RESTCONF protocols. The content of the configuration
template must be an instantiated chunk of data starting from any level node in the module hierarchies.

For example, {{temp-ex-interface}} provides an interface configuration template
that sets "mtu" as 1500 for ethernet interfaces:

~~~~
<templates>
  <template>
    <id>interface-type-mtu</id>
    <interface>
      <type>ianaift:ethernetCsmacd</type>
      <mtu>1500</mtu>
      <description>MTU value is set by template</description>
    </interface>
  </template>
</templates>
~~~~
{: #temp-ex-interface title="Example of An Interface template"}

The YANG data model of configuration templates is defined in {{template-yang}}.

### Templates with Regular Expressions

TBC

## Applying Templates {#inheriting-temp}

This document allows configuration templates to be inherited by
configuration nodes in the data tree at corresponding level.

If a configuration template is inherited by a node in the data tree, it acts as
if the configuration defined in the template is contained and is
merged with the configuration provided explicitly at the corresponding level in the data tree
with the explicitly provided configuration takes precedence.

If a configuration template is inherited by another new template,
the configuration of the new template is the merging result of configuration defined
in both templates with the new template takes precedence over its parent template.
This is useful when some additional configuration is intended to be defined on the
basis of the parent template.

Any modification to the parent template also applies where the template is inherited.


### The "stmt-extend" Metadata

Template inheritance is indicated by declaring the metadata object called "stmt-extend".

If the template is inherited by a node in the data tree, the metadata object is added
to that specific node.

If the template is inherited by other templates, the metadata object is added to
the node at corresponding level of the template contents.


The "stmt-extend" metadata MUST have only one value to specify the parent template
identifier that is inherited. The encoding of "stmt-extend" metadata object follows the way defined
in {{Section 5 of ?RFC7952}}.

For example, a client may configure physically present interfaces "eth0" and "eth1"
with the list node "interface" inheriting the template defined in {{temp-ex-interface}}:

~~~~
<interfaces xmlns:template="urn:ietf:params:xml:ns:yang:ietf-template">
  <interface template:stmt-extend="interface-type-mtu">
    <name>eth0</name>
  </interface>
  <interface template:stmt-extend="interface-type-mtu">
    <name>eth1</name>
  </interface>
</interfaces>
~~~~

And the above interface configuration renders the following expanded configuration:

~~~~
<interfaces>
  <interface>
    <name>eth0</name>
    <type>ianaift:ethernetCsmacd</type>
    <mtu>1500</mtu>
    <description>MTU value is set by template</description>
  </interface>
  <interface>
    <name>eth1</name>
    <type>ianaift:ethernetCsmacd</type>
    <mtu>1500</mtu>
    <description>MTU value is set by template</description>
  </interface>
</interfaces>
~~~~

### Template Precedence Rules

TBC

## Overriding Templates {#overriding-temp}

If there is some further configuration data that needs to be created but not included
in the parent template, it can be provided at the corresponding level when
inheriting the configuration template. For example, the client may want to define
another template and provide an additional "enabled" leaf value
on the basis of template defined in {{temp-ex-interface}}:

~~~~
<templates>
  <template>
    <id>interface-type-mtu-enabled</id>
    <interface xmlns:template="urn:ietf:params:xml:ns:yang:ietf-template"
               template:stmt-extend="interface-type-mtu">
      <enabled>true</enabled>
    </interface>
  </template>
</templates>
~~~~

And the above interface configuration defined in the template
"interface-type-mtu-enabled" renders the following expanded configuration:

~~~~
<interface>
  <type>ianaift:ethernetCsmacd</type>
  <mtu>1500</mtu>
  <description>MTU value is set by template</description>
  <enabled>true</enabled>
</interface>
~~~~

{{template-inherits}} provides more examples of inheriting an existing template by indicating
the "stmt-extend" metadata object.

It may be desired to override some configuration in an existing template when it is interited.
This may be achieved by directly editing the configuration template that is inherited,
however, the parent template may have also been inherited by other instance nodes or
templates, and direct modification of the parent template may yield unexpected results.

This document allows a configuration template to be overridden by
configuration explicitly provided by the client.

If there is some configuration values that need to be modified, the desired value
can be provided at the corresponding level when inheriting the configuation template.

For example, a client may configure physically present interfaces "eth0" and "eth1"
inheriting the template defined in {{temp-ex-interface}}, but the "mtu" value of "eth1"
needs to be 9122, and the "description" value also needs to be modified accordingly:

~~~~
<interfaces xmlns:template="urn:ietf:params:xml:ns:yang:ietf-template">
  <interface template:stmt-extend="interface-type-mtu">
    <name>eth0</name>
  </interface>
  <interface template:stmt-extend="interface-type-mtu">
    <name>eth1</name>
    <mtu>9122</mtu>
    <description>MTU value is set explicitly</description>
  </interface>
</interfaces>
~~~~

## Expanding Templates

TBC

## Validity of Templates

The contents of the template alone is not always sufficient to enforce the constraints
of the data model. Some constraints may depend on configuration outside of the
templates to satisfy, e.g., a list may contain a mandatory leaf node which is not
defined in the template but explicitly provided by the client. However, servers
should parse the template and enforce the constraints if it is possible during the
processing of template creation, e.g., servers may validate type constraints for the leaf,
including those defined in the type's "range", "length", and "pattern" properties.

That said, if a template is applied in the configuration data tree, the results of the template
configuration merging with configuration explicitly provided by the client MUST
always be valid, as defined in {{Section 8.1 of !RFC7950}}.

# Interaction with NMDA datastores

Some implementation may have predefined configuration templates for the convenience
of clients, which are present in \<system\> (if implemented, see {{?I-D.ietf-netmod-system-config}}).
In addition, clients can always define their own templates in \<running\>.
However, configuration template data defined by "ietf-template" YANG data model
should not be visible in \<operational\> until being inherited by a node in the data tree.

If a node in the data tree inherits a configuration template, the configuration
template does not expand in \<running\>, a read back of \<running\> returns what is
sent by the client with the "stmt-extend" metadata attached to the specific node.
Configuration template which is inherited or overridden by the node instance MUST be expanded in \<intended\>.

# Interaction with Non-NMDA datastores

TBC

# The "ietf-config-template" YANG Module {#template-yang}

## Data Model Overview

The following tree diagram {{?RFC8340}} illustrates the "ietf-config-template" module:

~~~~
{::include ./yang/ietf-template-tree.txt}
~~~~

> Editor's Note: Should the 'stmt-extend' and 'operation-tag' metadata annotations be defined here?

## YANG Module

~~~~
<CODE BEGINS> file "ietf-template@2025-03-28.yang"
{::include-fold ./yang/ietf-config-template.yang}
<CODE ENDS>
~~~~

# Security Considerations

TODO Security

# IANA Considerations

##  The "IETF XML" Registry

   This document registers the following URI in the "IETF XML Registry" {{!RFC3688}}.

~~~~
        URI: urn:ietf:params:xml:ns:yang:ietf-config-template
        Registrant Contact: The IESG.
        XML: N/A, the requested URI is an XML namespace.
~~~~

##  The "YANG Module Names" Registry

   This document registers the following YANG module in the "YANG Module Names"
   registry {{!RFC6020}}.

~~~~
        name:               ietf-config-template
        namespace:          urn:ietf:params:xml:ns:yang:ietf-config-template
        prefix:             template
        maintained by IANA? N
        reference:          RFC XXXX
~~~~


--- back

# Usage Examples {#appendix-network}

This section provides some examples to show the use of templates.
JSON encodings are used to not imply a preference in this document.
The fictional data model used throughout this section is shown as follows:

~~~~
{::include-fold ./yang/example-network-systime.yang}
~~~~

## Creating Templates {#template-creation}

The NTP configuration on multiple network devices may be consistent. To create a
template for NTP configuration, the following template configuration might be sent to a SDN controller:

~~~~
{
    "ietf-templates:templates": {
        "template": [
            {
                "id": "template-ntp",
                "content": {
                    "network-device": [
                        {
                            "ntp": {
                                "enabled": "true",
                                "server": [
                                    {
                                        "name": "ntp-server-1",
                                        "alias": [
                                            "primary"
                                        ],
                                        "address": "ntp.example-1.com"
                                    },
                                    {
                                        "name": "ntp-server-2",
                                        "alias": [
                                            "secondary"
                                        ],
                                        "address": "ntp.example-2.com"
                                    }
                                ]
                            }
                        }
                    ]
                }
            }
        ]
    }
}
~~~~

## Applying Templates {#template-inherits}

The operator may create another template with an additional NTP server instance
when inheriting the template created in {{template-creation}}. The configuration
is shown as follows:

~~~~
{
    "ietf-templates:templates": {
        "template": [
            {
                "id": "template-ntp2",
                "content": {
                    "network-device": [
                        {
                            "@": {
                                "ietf-template:stmt-extend": "template-ntp"
                            },
                            "ntp": {
                                "server": [
                                    {
                                        "name": "ntp-server-3",
                                        "alias": [
                                            "secondary"
                                        ],
                                        "address": "ntp.example-3.com"
                                    }
                                ]
                            }
                        }
                    ]
                }
            }
        ]
    }
}
~~~~

The configuration of template "template-ntp2" renders the following expanded configuration:

~~~~
{
    "ietf-templates:templates": {
        "template": [
            {
                "id": "template-ntp2",
                "content": {
                    "network-device": [
                        {
                            "ntp": {
                                "enabled": "true",
                                "server": [
                                    {
                                        "name": "ntp-server-1",
                                        "alias": [
                                            "primary"
                                        ],
                                        "address": "ntp.example-1.com",
                                        "prefer": true
                                    },
                                    {
                                        "name": "ntp-server-2",
                                        "alias": [
                                            "secondary"
                                        ],
                                        "address": "ntp.example-2.com"
                                    },
                                    {
                                        "name": "ntp-server-3",
                                        "alias": [
                                            "secondary"
                                        ],
                                        "address": "ntp.example-3.com"
                                    }
                                ]
                            }
                        }
                    ]
                }
            }
        ]
    }
}
~~~~

the following shows the network-level ntp configuration
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

And it renders the following expanded configuration:

~~~~
{
    "example-network-systime:network-device": [
        {
            "device-id": "ne-0",
            "ntp": {
                "enabled": "true",
                "server": [
                    {
                        "name": "ntp-server-1",
                        "alias": [
                            "primary"
                        ],
                        "address": "ntp.example-1.com"
                    },
                    {
                        "name": "ntp-server-2",
                        "alias": [
                            "secondary"
                        ],
                        "address": "ntp.example-2.com"
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
                        "name": "ntp-server-1",
                        "alias": [
                            "primary"
                        ],
                        "address": "ntp.example-1.com"
                    },
                    {
                        "name": "ntp-server-2",
                        "alias": [
                            "secondary"
                        ],
                        "address": "ntp.example-2.com"
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
                        "name": "ntp-server-1",
                        "alias": [
                            "primary"
                        ],
                        "address": "ntp.example-1.com"
                    },
                    {
                        "name": "ntp-server-2",
                        "alias": [
                            "secondary"
                        ],
                        "address": "ntp.example-2.com"
                    },
                    {
                        "name": "ntp-server-3",
                        "alias": [
                            "secondary"
                        ],
                        "address": "ntp.example-3.com"
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
                        "name": "ntp-server-1",
                        "alias": [
                            "primary"
                        ],
                        "address": "ntp.example-1.com"
                    },
                    {
                        "name": "ntp-server-2",
                        "alias": [
                            "secondary"
                        ],
                        "address": "ntp.example-2.com"
                    },
                    {
                        "name": "ntp-server-3",
                        "alias": [
                            "secondary"
                        ],
                        "address": "ntp.example-3.com"
                    }
                ]
            }
        }
    ]
}
~~~~

## Overriding Templates

The client may override the template created in {{template-creation}} to specify
the NTP server named "ntp-server-2" as the perferred one for device "ne-4":

~~~~
{
    "example-network-systime:network-device": [
        {
            "@": {
                "ietf-template:stmt-extend": "template-ntp"
            },
            "device-id": "ne-4",
            "server": [
                {
                    "name": "ntp-server-1",
                    "alias": [
                        "primary",
                        "secondary"
                    ],
                    "@alias": [
                        {
                            "ietf-template:operation-tag": "delete"
                        }
                    ],
                    "address": "ntp.example-1.com"
                },
                {
                    "@": {
                        "ietf-template:operation-tag": "position-first"
                    },
                    "name": "ntp-server-2",
                    "alias": [
                        "primary",
                        "secondary"
                    ],
                    "@alias": [
                        null,
                        {
                            "ietf-template:operation-tag": "delete"
                        }
                    ],
                    "address": "ntp.example-2.com"
                }
            ]
        }
    ]
}
~~~~

It is equivalent to the configuration as follows:

~~~~
{
    "example-network-systime:network-device": [
        {
            "device-id": "ne-4",
            "ntp": {
                "enabled": "true",
                "server": [
                    {
                        "name": "ntp-server-2",
                        "alias": [
                            "primary"
                        ],
                        "address": "ntp.example-2.com"
                    },
                    {
                        "name": "ntp-server-1",
                        "alias": [
                            "secondary"
                        ],
                        "address": "ntp.example-1.com"
                    }
                ]
            }
        }
    ]
}
~~~~

# Acknowledgments
{:numbered="false"}

The author would like to thank Lou Berger, Jason Sterne, Kent Watsen, and Robert
Wilton for comments and contributions made during interim meetings.

The author would like to acknowledge the following drafts and
presenters for kick-starting discussions on Yang Templates:

*  draft-ma-netmod-yang-config-template-00

*  draft-rajaram-netmod-yang-cfg-template-framework-00

*  Jan Lindblad
