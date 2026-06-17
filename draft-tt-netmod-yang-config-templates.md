---
title: "YANG Configuration Templates"
abbrev: "template"
category: std

docname: draft-tt-netmod-yang-config-templates-latest
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
   fullname: Robert Wills
   organization: Cisco
   role: editor
   country: United Kingdom
   email: rowills@cisco.com
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

  XSD-TYPES:
     title: "XML Schema Part 2: Datatypes Second Edition"
     author:
       -
         name: Paul V. Biron
       -
         name: Kaiser Permanente
       -
         name: Ashok Malhotra
     target: http://www.w3.org/TR/2004/REC-xmlschema-2-20041028
     date: false

--- abstract

   NETCONF and RESTCONF protocols provide programmatic interfaces for
   accessing configuration data modeled by YANG.  This document defines
   the use of a YANG-based configuration template mechanism whereby
   configuration data can be defined in one or more templates and
   applied repeatedly.  This avoids the redundant definition of
   identical configuration and ensures the consistency of it, thus
   allowing devices to be managed more conveniently and efficiently.

--- middle

# Introduction

   This document considers the case of a datastore that contains
   multiple subtrees with similar or identical nodes within them, such
   that the datastore contains repetitive data with limited variation.
   If a client has to repeatedly configure the same nodes for each
   subtree, this can become complex, error-prone, and masks the intent
   of the client.

   This document proposes a solution to improve this, called
   "Configuration Templates", that results in a smaller running
   datastore even when the configuration in \<running\> is large.

   A Configuration Template is a fragment of configuration that the
   device is instructed to replicate multiple times to generate copies
   of the configuration.  This allows repetitive subtrees of
   configuration to be written only once, in the template.  When needed,
   individual instantiations of a template can override the values of
   nodes, or add new instance-specific nodes.

   NMDA {{?RFC8342}} allows the configuration templates to be defined in
   \<running\> and expanded in \<intended\>, but it does not specify details
   about how configuration templates could be created and applied.

   This document defines the use of configuration templates in the
   context of YANG-driven network management protocols such as NETCONF
   {{!RFC6241}} and RESTCONF {{!RFC8040}}.  Configuration templates can be
   used with any YANG data model, this document doesn't make any
   assumption on the YANG data model design, i.e. it does not rely on a
   shared profile/group being defined in the YANG data model.


## Editorial Note (To be removed by RFC Editor)

Note to the RFC Editor: This section is to be removed prior to publication.

This document contains placeholder values that need to be replaced with finalized
values at the time of publication.  This note summarizes all of the
substitutions that are needed.  No other RFC Editor instructions are specified
elsewhere in this document.

Please apply the following replacements:

   * XXXX --> the assigned RFC number for this draft
   * 2025-05-28 --> the actual date of the publication of this document

# Conventions and Definitions

{::boilerplate bcp14-tagged}

The meanings of the symbols in tree diagrams are defined in
{{?RFC8340}}.

This document uses the YANG terminology defined in {{Section 3 of !RFC7950}}.

Besides, this document defines the following terminology:

Configuration Template:
: A chunk of reusable configuration data that
      could be applied to the configuration repeatedly, in order to
      simplify the delivery of network configuration and ensure the
      consistency of it.  A configuration template may also be called
      "template" or "YANG template" throughout this document.

# Requirements {#requirements}

This section describes the requirements that the Configuration
  Templates solution must satisfy.  These requirements were all
  discussed in the Interim Meetings, and a rough consensus was reached
  on each of them by the participants in the meetings.  A general theme
  of the Configuration Templates work is to come up with a "Minimal
  Viable Product" that is useful but not over-complicated.  More
  advanced features could be considered as extensions in later drafts.

## Defining and Managing Templates

Templates can be used with any YANG module.  They contain nodes of
  configuration data, and are stored persistently in the running
  datastore of the device.

  A client can view and manipulate a template, including the
  configuration inside it, by manipulating it in the \<running\>
  datastore.  In this sense, a template and its contents behaves like
  any other subtree of configuration.

## Applying Templates {#template-inherits}

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
particular subtree of the template takes effect for when the template
is applied.

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

# Configuration Template Solution

## Defining Templates {#define-templates}

A configuration template must first be defined before it can be applied (see {{inheriting-temp}}). The creation,
modification, and deletion of configuration templates is achieved by network
management operations via NETCONF or RESTCONF protocols. The contents of the configuration
template must be an instantiated chunk of data starting from any level node in the hierarchies of any YANG data model.

(Editor's note: more work may be needed here to ensure the template
is a valid subtree of config from a schema perspective.  This may
mean we need a way of saying where the root of the template is in the
schema, for example with a set of "outer" nodes with
operation="none").

The YANG data model of configuration templates is defined in {{template-yang}}.

### Template Definition with Pattern Matching {#regex}

To allow a single template to apply to multiple instances with similar naming conventions without explicit replication, pattern matching may be used within key leafs to restrict which list entries a template takes effect for. It is used to restrict the built-in type "string", or types derived from "string", to values that match the pattern.

Any regular expression pattern MUST conform to {{!RFC9485}}, which defines
a subset of XML Schema Definition (XSD) regular expressions {{XSD-TYPES}}.

 For example, {{regex-example}} provides an interface configuration template
 that sets "type" as ethernetCsmacd and "mtu" as 1500 for interfaces
 names match the pattern "eth.*", i.e., starting with the prefix "eth":

~~~~
<templates xmlns="urn:ietf:params:xml:ns:yang:ietf-config-template">
  <template>
    <id>ethernet-interface</id>
    <content>
      <interfaces xmlns="urn:example:interface">
        <interface>
          <name>eth.*</name>
          <type>ethernetCsmacd</type>
          <mtu>1500</mtu>
        </interface>
      </interfaces>
    </content>
  </template>
</templates>
~~~~
{: #regex-example title="Example of An Interface template" artwork-align="center"}

## Applying Templates {#inheriting-temp}

For each configuration node in the \<running\> datastore, one or more
templates can be applied.  This causes configuration from the
templates to be combined with child configuration in the \<running\>
datastore to produce a final set of \<intended\> configuration that
will be used by the device.


### The "apply-templates" Metadata

Template application is indicated using the "apply-templates"
metadata.  The value of this is a list of space-separated template
identifiers.  If the template is applied to a node in the data tree,
the metadata object is added to that specific node.

The encoding of "apply-templates" metadata object follows the way defined
in {{Section 5 of ?RFC7952}}.

For example, the following interface configuration may be provided
with the container node "interfaces" applying the template defined in
{{regex-example}}:

~~~~
    <interfaces xmlns="urn:example:interface"
      xmlns:ct="urn:ietf:params:xml:ns:yang:ietf-config-template"
      ct:apply-templates="ethernet-interface">
      <interface>
        <name>loopback0</name>
      </interface>
      <interface>
        <name>eth0</name>
      </interface>
      <interface>
        <name>eth1</name>
      </interface>
    </interfaces>
~~~~

And the above interface configuration renders the following expanded configuration:

~~~~
    <interfaces xmlns="urn:example:interface">
      <interface>
        <name>loopback0</name>
      </interface>
      <interface>
        <name>eth0</name>
        <type>ethernetCsmacd</type>
        <mtu>1500</mtu>
      </interface>
      <interface>
        <name>eth1</name>
        <type>ethernetCsmacd</type>
        <mtu>1500</mtu>
      </interface>
    </interfaces>
~~~~

### Creating, editing and deleting the "apply-templates" metadata

The apply-templates metadata can be modified by the client by
specifying it as an attribute in an \<edit-config\> request.  There are
three cases:

*  The apply-templates attribute is specified and the value is non-
   empty (i.e. a list of templates to apply to the node).  The apply-
   templates metadata is changed to match the value in the request.

> Editor's Note: What if a specific node has some templates applied, and another \<edit-config\> provides another set of values of apply-template? It is a merge or full replace? Should this whole solution be combined with NETCONF "operation" attribute?

*  The apply-templates attribute is specified and the value is the
   empty string.  The apply-templates metadata is removed and thus no
   templates are applied to the node.

*  The apply-templates attribute not specified.  The apply-templates
   metadata currently present on the node (if any) is unchanged.

For example, this request creates a single loopback0 interface and
applies template t1 to the interfaces container:

~~~~
    <edit-config>
      ...
      <config>
        <interfaces xmlns="urn:example:interface"
                    ct:apply-templates="t1">
          <interface>
            <name>loopback0</name>
          </interface>
        </interfaces>
      </config>
    </edit-config>
~~~~

This request also applies template t2 to the interfaces container:

~~~~
    <edit-config>
      ...
      <config>
        <interfaces xmlns="urn:example:interface"
                    ct:apply-templates="t1 t2" />
      </config>
    </edit-config>
~~~~

After this request, \<running\> is as follows:

~~~~
    <interfaces xmlns="urn:example:interface"
                ct:apply-templates="t1 t1">
      <interface>
        <name>loopback0</name>
      </interface>
    </interfaces>
~~~~

This request adds a new interface list entry, and leaves the applied
templates unchanged:

~~~~
    <edit-config>
      ...
      <config>
        <interfaces xmlns="urn:example:interface">
          <interface>
            <name>eth0</name>
          </interface>
        </interfaces>
      </config>
    </edit-config>
~~~~

After this request, \<running\> is as follows:

~~~~
    <interfaces xmlns="urn:example:interface"
                ct:apply-templates="t1 t1">
      <interface>
        <name>loopback0</name>
      </interface>
      <interface>
        <name>eth0</name>
      </interface>
    </interfaces>
~~~~

Finally, this request deletes all the templates, and leaves the list
entries unchanged:

~~~~
    <edit-config>
      ...
      <config>
        <interfaces xmlns="urn:example:interface"
                    ct:apply-templates="" />
        </interfaces>
      </config>
    </edit-config>
~~~~

After this request, \<running\> is as follows:

~~~~
    <interfaces xmlns="urn:example:interface">
      <interface>
        <name>loopback0</name>
      </interface>
      <interface>
        <name>eth0</name>
      </interface>
    </interfaces>
~~~~

## Overriding Templates {#overriding-temp}

The client may want to to override some configuration in a template
 when it is applied to a particular node in \<running\>.  The client can
 achieve this by providing the desired value at the corresponding
 level when applying the template.  Configuration explicitly provided
 by the client always takes precedence over the same node defined in
 template.

 A template node can be overriden by having its value changed, but it
 can't be deleted.

 As an example of overriding a node in a template, a client may
 configure physically present interfaces "eth0" and "eth1" inheriting
 the template defined in Figure 1, but the "mtu" value of "eth1" needs
 to be 9122:

~~~~
    <interfaces xmlns="urn:example:interface"
      xmlns:ct="urn:ietf:params:xml:ns:yang:ietf-config-template"
      ct:apply-templates="ethernet-interface">
      <interface>
        <name>loopback0</name>
      </interface>
      <interface>
        <name>eth0</name>
      </interface>
      <interface>
        <name>eth1</name>
        <mtu>9122</mtu>
      </interface>
    </interfaces>
~~~~

 And the above interface configuration renders the following expanded
 configuration:

~~~~
    <interfaces xmlns="urn:example:interface">
      <interface>
        <name>loopback0</name>
      </interface>
      <interface>
        <name>eth0</name>
        <type>ethernetCsmacd</type>
        <mtu>1500</mtu>
      </interface>
      <interface>
        <name>eth1</name>
        <type>ethernetCsmacd</type>
        <mtu>9122</mtu>
      </interface>
    </interfaces>
~~~~

## Expanding Templates {#expand-templates}

When a configuration template is applied to a node in the data tree,
it acts as if the configuration defined in the template is merged
with the configuration provided explicitly at the corresponding level
in the data tree, with the explicitly provided configuration taking
precedence.

the process of expanding templates to derive \<intended\> is deterministic and depends solely on the contents of \<running\>.
The process rules are as follows:

*  The value of a node in the \<intended\> configuration is determined
   by using precedence to decide where to take the value from.

*  Non-template config always has the highest precedence.

*  When templates are applied to multiple ancestors, the innermost
   ancestor takes precedence.

*  When multiple templates are applied to a particular node, the
   order of application (as indicated by the client when applying the
   templates) determines the precedence within that node.

If a client has knowledge of the complete contents of \<running\>,
it can calculate the exact result of template expansion, independent of the server's operational state.

Whenever the contents of a template is updated in \<running\>, the
result of expanding out the template appears in \<intended\> and takes
effect on the device.

## Deletion of Templates

After a template has been applied to a node in the data tree,
the template configuration itself MAY be allowed to be deleted
while the expanded configuration still remains in the intended datastore.

## Validity of Templates

The contents of the template alone is not always sufficient to
enforce the constraints of the data model.  Some constraints may
depend on configuration outside of the templates to satisfy, e.g., a
list may contain a mandatory leaf node which is not defined in the
template but explicitly provided by the client.  However, servers
SHOULD parse the template and enforce the constraints if it is
possible during the processing of template creation, e.g., servers
may validate type constraints for the leaf, including those defined
in the type's "range", "length", and "pattern" properties. Implementations
may also consider using mechanism defined in {{?I-D.ietf-netmod-yang-anydata-validation}} to validate anydata.

> Editor's Note: Should the validity of template configuration be mandatory or optional?

That said, if a template is applied in the configuration data tree,
the results of the template configuration merging with configuration
explicitly provided by the client MUST always be valid, as defined in
{{Section 8.1 of !RFC7950}}.

# Interaction with NMDA datastores {#interact-NMDA}

Some implementations may have predefined configuration templates for the convenience
of clients, which are present in \<system\> (if implemented, see {{?I-D.ietf-netmod-system-config}}).
In addition, clients can always define their own templates in \<running\>.
However, configuration template data defined by "ietf-config-template" YANG data model
should not be visible in \<operational\> until being inherited by a node in the data tree.

If a node in the data tree applies a configuration template, the configuration
template does not expand in \<running\>. A read of \<running\> returns what is
sent by the client with the "apply-templates" metadata attached to the specific node.
A configuration template which is inherited or overridden by the node instance MUST be expanded in \<intended\>.

# Interaction with Non-NMDA datastores {#interact-non-NMDA}

TBC

# The "ietf-config-template" YANG Module {#template-yang}

## Data Model Overview

The following tree diagram {{?RFC8340}} illustrates the "ietf-config-template" module:

~~~~
{::include ./yang/ietf-template-tree.txt}
~~~~

> Editor's Note: Should we use the RFC7952 metadata annotation for the 'apply-templates' metadata here?

> Editor's Note: the current definition of template configuration
      uses anydata, but this may not be able to be validated at template
      definition time because anydata is opaque.

## YANG Module

~~~~
<CODE BEGINS> file "ietf-template@2025-05-28.yang"
{::include-fold ./yang/ietf-config-template.yang}
<CODE ENDS>
~~~~


# Operational Considerations {#operational-consideration}

Implementations MAY restrict the applications of configuration templates to some specific nodes in the YANG data tree. Restrictions should be applied consistently across all client operations. Any attempts to apply a template to a restricted node will be rejected. Implementations are recommended to expose the list of configuration nodes that do not support template application, any mechanisms to achieve this are outside the scope of this document.

Configuration templates are designed to remain unexpanded in \<running\>. This ensures storage efficiency and preserves the client's control over \<running\>, i.e., reads of \<running\> returns the client-submitted configuration with the "apply-templates" metadata attached to target nodes. Any configuration template that is applied in the data tree MUST be expanded in \<intended\>, which holds a merged result of template expansion and configuration explicitly provided by clients.

Implementations MAY differ in whether the configuration templates themselves appear in \<intended\>, independent of whether the templates are applied. Implementations MAY also support conditional visibility, where templates appear in \<intended\> only when they are applied by at least one node via the "apply-templates" annotation. Regardless of the approach chosen, implementations MUST ensure the behavior is consistent and deterministic, and SHOULD be documented to allow clients to rely on predictable operational behaviors.


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
        prefix:             ct
        maintained by IANA? N
        reference:          RFC XXXX
~~~~


--- back

# Requirement Implementation Status

Note to the RFC Editor: Please remove this section before publication.

This appendix aims to track which of identified requirements have been addressed in the current version, and, where applicable, how they are fulfilled by the proposed mechanism.

| Requirement | Fulfilled | Requirement Description |
| R1: Allowed Multiple templates to be applied at a single node | Y | see {{inheriting-temp}} |
| R2: Templates must work with any YANG module | Y | see {{define-templates}} |
| R3: Templates must be validated when defined | N | Needs further discussion, see Editor's note from {{define-templates}} |
| R4: Local-config overrides template-config | Y | see {{overriding-temp}} |
| R5: Living template: modified template data gets expanded for all consumers | Y | see {{expand-templates}} |
| R6: Support basic programmatic elements in templates | N | Seems to add some complexity |
| R7: Allow a server to constrain which nodes can be templates consumer | Y | See {{operational-consideration}} |
| R8: Configuration with both expanded and unexpanded templates is able to be returned | Y | see {{interact-NMDA}} and {{operational-consideration}} |
| R9: \<running\> contains the unexpanded template | Y | see {{interact-NMDA}}, also stated explicitly in {{operational-consideration}} |
| R10: \<intended\> contains the expanded template | Y | see {{interact-NMDA}}, also stated explicitly in {{operational-consideration}} |
| R11: Enables off-box template expansion of \<running\> | Y | see {{expand-templates}} |
| R12: Support limited regex in templates | Y | see {{regex}} |
| R13: Have a precedence rule when multiple templates are applied at a single node | Y | See {{expand-templates}} |
| R14: The innermost template takes precedence when templates are applied at multiple ancestor nodes | Y | See {{expand-templates}} |
| R15: Enable non-NMDA servers to return the expanded data | N | have a dedicated section ({{interact-non-NMDA}}) for this, but empty now |
| Not discussed: R16: exclude templates applied at ancestor nodes | N | Seems to add some complexity, needs further discussion |
| Not discussed: R17: Annotations to determine which template a node was applied from | N | Needs further discussion |


<!--
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

## Applying Templates

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
-->

# Acknowledgments
{:numbered="false"}

The author would like to thank Lou Berger, Jason Sterne, Kent Watsen, and Robert
Wilton for comments and contributions made during interim meetings.

The author would like to acknowledge the following drafts and
presenters for kick-starting discussions on Yang Templates:

*  draft-ma-netmod-yang-config-template-00

*  draft-rajaram-netmod-yang-cfg-template-framework-00

*  draft-wills-netmod-yang-templates-00

*  Jan Lindblad
