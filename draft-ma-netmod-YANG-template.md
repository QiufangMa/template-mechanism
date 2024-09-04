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

normative:

informative:


--- abstract

NETCONF and RESTCONF protocols provide programmatic operation interfaces for accessing
configuration data modeled by YANG. This document defines the use of YANG-based
configuration template mechanism so that the configuration data could be defined as template
and applied repeatedly to avoid the redundant definition of identical Configuration
and ensure consistency of it.

--- middle

# Introduction

It is not unusual for the YANG data model {{!RFC7950}} to define some shared profiles that could
be referenced in order to simplify the configuration of network services or functionalities.
For example, {{?I-D.ietf-opsawg-ntw-attachment-circuit}} defines a set of profiles
at the network level which could be referred to under the node level to factorize
some common configuration shared by a group of attachment circuits (ACs).

However, it is not trivial to always take care of the definition of shared profiles/
policies/templates during the design of every data model.
There is a desire to make use of common YANG-based templates without relying on
specific definition in YANG data models.

NMDA {{?RFC8342}} allows the configuration templates to be defined in \<running\>
and expanded in \<intended\>, but it does not specify details about how configuration
templates could be created and applied.

This document defines the use of configuration templates in the context of YANG-driven
network management protocols such as NETCONF {{!RFC6241}} and RESTCONF {{!RFC8040}}.
By defining a common set of nodes as a configuration template and applying the
configuration template repeatedly, it avoids the redundant definition of identical
configuration and also ensures consistency of it. Configuration template could
be used based on any existing YANG data models, this document doesn't make any
assumption on the YANG data model design, i.e., it does not rely on the shared profile/group
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
  ensure the consistency of it. A configuration template can also be called
  "hierarchical template", or "template" for short.

inherited template:
: A configuration template that is applied in the configuration data tree or
  reused by other templates.

parent template:
: A configuration template that is an inherited template.

# Hierarchical Template Overview

A configuration template must first be defined before it can be inherited {{inheriting-temp}}. The creation,
modification, and deletion of configuration templates are achieved by network
management operations via NETCONF or RESTCONF protocols. The content of the configuration
template must be an instantiated chunk of data starting from at least one top-level node in the module hierarchies.

For example, {{temp-ex-interface}} provides an interface configuration template
that sets "mtu" as 1500 for ethernet interfaces:

~~~~
<templates>
  <template>
    <id>interface-type-mtu</id>
    <interfaces>
      <interface>
        <type>ianaift:ethernetCsmacd</type>
        <mtu>1500</mtu>
        <description>MTU value is set by template</description>
      </interface>
    </interfaces>
  </template>
</templates>
~~~~
{: #temp-ex-interface title="Example of An Interface template"}

The YANG data model of configuration templates is defined in {{template-yang}}.

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

# Inheriting Templates {#inheriting-temp}

This document allows configuration templates to be inherited by top-level
configuration nodes in the data tree or new templates. A node inherits at most
one configuration template.

If a configuration template is inherited by a node in the data tree, it acts as
if the configuration defined in the template is contained and is
merged with the configuration provided explicitly at the corresponding level in the data tree
with the explicitly provided configuration takes precedence.

If a configuration template is inherited by another new template,
the configuration of the new template is the merging result of configuration defined
in both templates with the new template takes precedence over its parent template.
This is useful when some additional configuration is intended to be defined on the
basis of the parent template.

Any modification to the parent template also applies somewhere inherits the template.
Care MUST be taken when making changes to the parent templates.

## The "stmt-extend" Metadata

Template inheritance is flagged by declaring the metadata object called "stmt-extend".

If the template is inherited by a top-level node in the data tree, the metadata object is added
to that specific node. Server MUST igore any metadata objects added to the node
that is not a top-level node.

If the template is inherited by other templates, the metadata object is added to
the top-level node of the template contents.


The "stmt-extend" metadata MUST have only one value to specify the parent template
identifier that is inherited. The encoding of "stmt-extend" metadata object follows the way defined
in {{Section 5 of ?RFC7952}}.

For example, a client may configure physically present interfaces "eth0" and "eth1"
with the container node "interfaces" inheriting the template defined in {{temp-ex-interface}}:

~~~~
<interfaces xmlns:template="urn:ietf:params:xml:ns:yang:ietf-template"
  template:stmt-extend="interface-type-mtu">
  <interface>
    <name>eth0</name>
  </interface>
  <interface>
    <name>eth1</name>
  </interface>
</interfaces>
~~~~

And the above interface configuration is equivalent to the following:

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

## Template Extension

If there is some further configuration data that needs to be created but not included
in the parent template, it can be provided at the corresponding level when
inheriting the configuration template. For example, the client may want to define
another template and provide an additional "enabled" leaf value
on the basis of template defined in {{temp-ex-interface}}:

~~~~
<templates>
  <template>
    <id>interface-type-mtu-enabled</id>
    <interfaces xmlns:template="urn:ietf:params:xml:ns:yang:ietf-template"
      template:stmt-extend="interface-type-mtu">
      <interface>
        <enabled>true</enabled>
      </interface>
    </interfaces>
  </template>
</templates>
~~~~

And the above interface configuration defined in the template
"interface-type-mtu-enabled" is equivalent to the following:

~~~~
<interfaces>
  <interface>
    <type>ianaift:ethernetCsmacd</type>
    <mtu>1500</mtu>
    <description>MTU value is set by template</description>
    <enabled>true</enabled>
  </interface>
</interfaces>
~~~~

{{template-inherits}} provides more examples of inheriting an existing template by indicating
the "stmt-extend" metadata object.

# Overriding Templates {#overriding-temp}

It may be desired to override some configuration in an existing template when it is interited.
This may be achieved by directly editing the configuration template that is inherited,
however, the parent template may have also been inherited by other instance nodes or
templates, and direct modification of the parent template may yield unexpected results.

This document allows a configuration template to be overridden by other templates
or configuration explicitly provided by the client.

## Modification

If there is some configuration values that need to be modified, the desired value
can be provided at the corresponding level when inheriting the configuation template.

For example, a client may configure physically present interfaces "eth0" and "eth1"
inheriting the template defined in {{temp-ex-interface}}, but the "mtu" value of "eth1"
needs to be 9122, and the "description" value also needs to be modified accordingly:

~~~~
<interfaces xmlns:template="urn:ietf:params:xml:ns:yang:ietf-template"
  template:stmt-extend="interface-type-mtu">
  <interface>
    <name>eth0</name>
  </interface>
  <interface>
    <name>eth1</name>
    <mtu>9122</mtu>
    <description>MTU value is set explicitly</description>
  </interface>
</interfaces>
~~~~


## Deletion

The deletion of configuration data is flagged by declaring the metadata object
called "operation-tag" with a value "delete". If some node instance defined in the configuration template
is intended to be deleted in the configuration explicitly by the client or by new
templates, the metadata object is added to that specific node. Servers MUST ignore this
metadata if the configuration identified currently does not exist in the configuration
template.

The "operation-tag" metadata MUST have only one value to specify the intended operation.
The encoding of "operation-tag" metadata object follows the way defined in {{Section 5 of ?RFC7952}}.

For example, a client may want to delete the "description" node instance defined
in {{temp-ex-interface}} for interface "eth1", and the following shows the example
configuration:

~~~~
<interfaces xmlns:template="urn:ietf:params:xml:ns:yang:ietf-template"
  template:stmt-extend="interface-type-mtu">
  <interface>
    <name>eth0</name>
  </interface>
  <interface>
    <name>eth1</name>
    <mtu>9122</mtu>
    <description template:operation-tag="delete">MTU value is set by template</description>
  </interface>
</interfaces>
~~~~

And it is equivalent to the following:

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
    <mtu>9122</mtu>
  </interface>
</interfaces>
~~~~

## List/leaf-list Reordering

There may be a desire to reorder some existing list/leaf-list entries defined in an
existing template, or specify the ordering of new instances when they are created.
It only applies to lists and leaf-lists indicated with the "ordered-by user"
statement.

The reordering of list/leaf-list entry data is flagged by declaring the metadata
object called "operation-tag" with different values. The "operation-tag" used
for reordering has one of the following values:

position-first: The attached entry is positioned as the first entry in the list/leaf-list.
  There must be no more than one entry that is annotated as "position-first" within the same operation request.

position-last: The attached entry is positioned as the last entry in the list/leaf-list.
  There must be no more than one entry that is annotated as "position-last" within the same operation request.

position-before: The attached entry is positioned before some other specified
  entries in the list/leaf-list. This value MUST be followed by one or more
  space-seperated key values to indicate the entries that the attached entry is
  just positioned before. The "position-before" metadata and key values are
  seperated by colon ":". There must not be any other entries that use
  "position-before" to target the entry attached as "position-first" within the
  same operation rquest.

: position-after: The attached entry is positioned after some other specified
  entries in the list/leaf-list. This value MUST be followed by one or more
  space-seperated key values to indicate the entries that the attached entry is
  just positioned after. The "position-after" metadata and key values are
  seperated by colon ":". There must not be any other entries that use
  "position-after" to target the entry attached as "position-last" within the
  same operation request.

For example, the following template provides the user-ordered ACL configuration:

~~~~
<templates>
  <template>
    <id>ACL-rules-template</id>
    <acls>
      <acl>
        <name>allow-access-from-tcp</name>
        <matches>
          <protocol>tcp</protocol>
        </matches>
        <action>permit</action>
      </acl>
      <acl>
        <name>allow-access-from-udp</name>
        <matches>
          <protocol>udp</protocol>
        </matches>
        <action>permit</action>
      </acl>
    </acls>
  </template>
</templates>
~~~~

If the client wants to create another template with an additional ACL entry named "deny-traffic-from-ftp"
ordered as the first entry and ACL entry named "allow-access-from-tcp" as the last one:

~~~~
<templates>
  <template>
    <id>ACL-rules-template2</id>
    <acls xmlns:template="urn:ietf:params:xml:ns:yang:ietf-template"
      template:stmt-extend="acl-rule-template">
      <acl template:operation-tag="position-first">
        <name>deny-traffic-from-ftp</name>
        <matches>
          <protocol>ftp</protocol>
        </matches>
      </acl>
      <acl template:operation-tag="position-last">
        <name>allow-access-from-tcp</name>
      </acl>
    </acls>
  </template>
</templates>
~~~~

The client may also deliver the configuration defined in template
"acl-rule-template" with some other additional ACL entries that is properly ordered:

~~~~
<acls xmlns:template="urn:ietf:params:xml:ns:yang:ietf-template"
  template:stmt-extend="acl-rule-template">
  <acl template:operation-tag="position-before:\
    'allow-access-from-tcp allow-access-from-udp'">
    <name>deny-access-to-ipv4</name>
    <matches>
      <destination-ipv4-network>192.0.2.0/24</destination-ipv4-network>
    </matches>
  </acl>
  <acl template:operation-tag="position-after:'deny-access-to-ipv4'">
    <name>deny-access-to-ipv6</name>
    <matches>
      <destination-ipv6-network>2001:db8::/32</destination-ipv6-network>
    </matches>
  </acl>
</acls>
~~~~

The applied ACL configuration is equivalent to the following:

~~~~
<acls>
  <acl>
    <name>deny-access-to-ipv4</name>
    <matches>
      <destination-ipv4-network>192.0.2.0/24</destination-ipv4-network>
    </matches>
  </acl>
  <acl>
    <name>deny-access-to-ipv6</name>
    <matches>
      <destination-ipv6-network>2001:db8::/32</destination-ipv6-network>
    </matches>
  </acl>
  <acl>
    <name>allow-access-from-tcp</name>
    <matches>
      <protocol>tcp</protocol>
    </matches>
    <action>permit</action>
  </acl>
  <acl>
    <name>allow-access-from-udp</name>
    <matches>
      <protocol>udp</protocol>
    </matches>
    <action>permit</action>
  </acl>
</acls>
~~~~

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

> Editor's Note: The read-back of \<running\> might break legacy clients doesn't
understand template?

# Different Levels of Templates

The configuration templates may be defined at different levels, depending on where it is used and maintained.
For exmaple, A network-level template maintained by the software-defined networking
(SDN) {{?RFC7149}} {{?RFC7426}} controller defines configuration that may be shared
by multiple network devices. While device-level template maintained by the network
element defines configuration that can only be applied to specific network devices.
Refer to {{appendix-network}} for examples of network-level templates.

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

## Inheriting Templates {#template-inherits}

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

The configuration of template "template-ntp2" is equivalent to the following:

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

And it is equivalent to the following configuration:

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

TODO acknowledge.
