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
   fullname: Kent Watsen
   organization: Watsen Networks
   email: kent+ietf@watsen.net
-
   fullname: Qiufang Ma
   organization: Huawei
   street: 101 Software Avenue, Yuhua District
   city: Jiangsu
   code: 210012
   country: China
   email: maqiufang1@huawei.com
-
   fullname: Deepak Rajaram
   organization: Nokia
   country: India
   email: deepak.rajaram@nokia.com

contributor:
-
   fullname: Robert Wills
   organization: Cisco
   country: United Kingdom
   email: rowills@cisco.com
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

   This document defines a YANG-based configuration template mechanism whereby
   configuration data can be defined in one or more templates and
   applied repeatedly.  This avoids the redundant definition of
   identical configuration and ensures the consistency of it, thus
   allowing configuration data to be managed more conveniently and efficiently.

--- middle

# Introduction

   This document defines the "template" mechanism mentioned but
   not defined in Network Management Datastore Architecture (NMDA) {{?RFC8342}}.

   Templates enable repetitive configuration to be factored out into
   a template and subsequently applied wherever the configuration
   is needed.  This avoids the redundant definition of identical
   configuration and ensures the consistency of it, thus allowing
   configuration data to be managed more conveniently and efficiently.

   By examnple, an network management system (NMS) may manage many
   devices.  Devices may be come from different vendors, each of
   which may have multiple types of devices (router, firewall, etc.),
   though sharing a common operating system.  Further, each type of
   device may have different models (e.g., fw-100, fw-1000, etc.).
   In this case, common "fw-100" configuration could be put into
   a template called "common-fw-100-template", which itself inherits
   from a template called "common-fw-template", which itself inherits
   from a template called "common-vendor-template".  Similarly, a
   "common-fw-1000-template" could inherit from "common-fw-template"
   and, likewise, a "common-rtr-template" could inherit from the
   "common-vendor-template".

   Templates are mostly for humans, but are still important in some
   cases when the configuration of a device is fully automated.
   Specifically, when provided templates, a device can optimize
   its memory, enabling higher performance and scability.

   The solution presented in this document supports both servers
   that do and do not support NMDA.  For server's that support NMDA,
   the solution is more complete, as templates may be defined in the
   \<system\> datastore defined in {{?I-D.ietf-netmod-system-config}},
   and the \<intended\> datastore always returns the configuration
   with the templates expanded.  For server's that do not support
   NMDA, a "with-templates-expanded" parameter may be passed by a
   client when fetching configuration.

   Configuration templates can be used with any YANG data model,
   including those defined with augmentations and/or deviations.


## Editorial Note (To be removed by RFC Editor)

Note to the RFC Editor: This section is to be removed prior to publication.

This document contains placeholder values that need to be replaced with finalized
values at the time of publication.  This note summarizes all of the
substitutions that are needed.  No other RFC Editor instructions are specified
elsewhere in this document.

Please apply the following replacements:

   * XXXX --> the assigned RFC number for this draft
   * 2026-07-03 --> the actual date of the publication of this document

# Conventions and Definitions

{::boilerplate bcp14-tagged}

The meanings of the symbols in tree diagrams are defined in
{{?RFC8340}}.

This document uses the terminology defined in {{Section 3 of !RFC7950}} and {{Section 3 of !RFC8342}}.

This document uses the following terminology in {{!RFC6241}}:

 * configuration data

Besides, this document defines the following terminology:

configuration template:
: A snippet of configuration data that may be applied to the
  configuration repeatedly, in order to simplify the delivery
  of network configuration and ensure the consistency of it.
  A configuration template is referred to interchangeably as
  "template" or "YANG template" throughout this document.

# Requirements {#requirements}

This section describes requirements that the configuration
  templates solution must satisfy. A general theme
  of the configuration template work is to come up with a "Minimal
  Viable Product (MVP)" that delivers a baseline solution with essential functionality but avoids excessive complexity. More
  advanced features could be considered as extensions in future work.

## Defining and Managing Templates

  Templates can be defined with any YANG module. They contain nodes of
  configuration data, and are stored in the running
  datastore of the server after creation.

  A client can view and manipulate a template in \<running\>. System may generate a template in \<system\> ({{?I-D.ietf-netmod-system-config}}).
  In this sense, a template and its contents behave like
  any other configuration data.

## Applying Templates {#template-inherits}

A template can be applied to zero or more nodes in the running
  datastore.  Each node can have zero or more templates applied to it,
  and the order specified by the client determines the precedence with which templates are applied within that node.

  Templates can be applied at multiple nodes in the hierachy. When viewing the contents of \<running\>, there is a mechanism to see
  which templates have been applied to each node, and in which order.

## Producing the Intended Datastore

The server's intended datastore is the result of combining all the
applications of templates together with non-template configuration (i.e., configuration explicitly created by clients rather than derived from applied templates) in \<running\> and \<system\> (see {{?I-D.ietf-netmod-system-config}}).  This is called template expansion.

The intended configuration inside a subtree is the result of taking
the relevant contents of every template applied to the subtree's root
node and its ancestors, and combining it with the (non-template) data
nodes inside the subtree.

A node inside a subtree may be present in multiple templates that
have been applied, and/or it may be present as non-template configuration
inside the subtree. The requirements for template expansion are as follows:

*  The value of a node in \<intended\> is determined
   by using precedence rule to decide where to take the value from.

*  Non-template configuration always has the highest precedence.

*  When templates are applied to multiple ancestors, the innermost
   ancestor takes precedence.

*  When multiple templates are applied to a particular node, the
   order of application (as indicated by the client when applying the
   templates) determines the precedence within that node.

Whenever the contents of a template is updated in \<running\> or \<system\>, the
result of template expansion appears in \<intended\>.

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

If the client knows the complete contents of \<running\> and \<system\>, which include non-
template configuration, template definitions and template applications, the client must be able to calculate the result of template
expansion, i.e., the contents of \<intended\>.

In other words, the outcome of template expansion depends solely on
the contents of running and system datastores.

# Configuration Template Solution

## Defining Templates {#define-templates}

A configuration template must first be defined before it can be applied (see {{inheriting-temp}}). The creation,
modification, and deletion of configuration templates are achieved by network
management operations via NETCONF or RESTCONF protocols. The contents of the configuration
template must be an instantiated chunk of data starting from any level node in the hierarchies of any YANG data model.

For example, {{base-template}} provides an interface configuration template named "base-interface":

~~~~
<templates xmlns="urn:ietf:params:xml:ns:yang:ietf-config-template">
 <template>
   <id>base-interface</id>
   <content>
     <interfaces xmlns="urn:example:interface">
       <interface>
         <enabled>true</enabled>
         <mtu>65536</mtu>
         <description>default provisioned interface</description>
       </interface>
     </interfaces>
   </content>
 </template>
</templates>
~~~~
{: #base-template title="Example of An Interface Template" artwork-align="center"}

The YANG data model of configuration templates is defined in {{template-yang}}.

### Template Definition with Pattern Matching {#regex}

To allow a single template to apply to multiple instances with similar naming conventions without explicit replication, a regular expression string may be used within key leafs to restrict which list entries a template takes effect for. It MUST NOT be used on any nodes other than a list key with built-in type "string", or types derived from "string".

Any regular expression pattern MUST conform to {{!RFC9485}}, which defines
a subset of XML Schema Definition (XSD) regular expressions {{XSD-TYPES}}.

 For example, {{regex-example}} provides an interface configuration template
 that sets "type" as ethernetCsmacd and "mtu" as 1500 for all interfaces
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
          <description>default provisioned ethernet interface</description>
        </interface>
      </interfaces>
    </content>
  </template>
</templates>
~~~~
{: #regex-example title="Example of An Interface Template with Pattern Matching" artwork-align="center"}

## Applying Templates {#inheriting-temp}

For each configuration node, including container, list, anydata, anyxml, leaf-list, and leaf, one or more
templates can be applied. This causes configuration from one or more
templates to be merged with explicitly provided configuration data
to produce a final set of configuration that is intended to be applied by the server.
Any update to the applied templates will be reflected in the merging result.

### The "apply-templates" Metadata {#apply-templates}

Template application is indicated using the "apply-templates"
metadata annotation {{?RFC7952}}.  The value of this is a list of space-separated template
identifiers. The order of appearance of the template identifiers in the list determines their precedence when producing the merging result. If the template is applied to a node in the data tree,
the metadata object is added to that specific node.

The encoding of "apply-templates" metadata object follows the way defined
in {{Section 5 of ?RFC7952}}.

For example, {{application-example}} provides the interface configuration
with the container node "interfaces" applying the templates "ethernet-interface" and "base-interface", defined in
{{regex-example}} and {{base-template}} respectively:

~~~~
    <interfaces xmlns="urn:example:interface"
      xmlns:ct="urn:ietf:params:xml:ns:yang:ietf-config-template"
      ct:apply-templates="ethernet-interface base-interface">
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
{: #application-example title="An Example of Applying Templates" artwork-align="center"}

And the above interface configuration renders the expanded configuration shown in {{expansion-result-1}}:

~~~~
    <interfaces xmlns="urn:example:interface">
      <interface>
        <name>loopback0</name>
        <enabled>true</enabled>
        <mtu>65536</mtu>
        <description>default provisioned interface</description>
      </interface>
      <interface>
        <name>eth0</name>
        <enabled>true</enabled>
        <type>ethernetCsmacd</type>
        <mtu>1500</mtu>
        <description>default provisioned ethernet interface</description>
      </interface>
      <interface>
        <name>eth1</name>
        <enabled>true</enabled>
        <type>ethernetCsmacd</type>
        <mtu>1500</mtu>
        <description>default provisioned ethernet interface</description>
      </interface>
    </interfaces>
~~~~
{: #expansion-result-1 title="Template Expansion" artwork-align="center"}

### Creating, editing and deleting the "apply-templates" metadata

The "apply-templates" metadata annotation may be modified by the client by
specifying a different value in subsequent operations. Any modification to this annotation MUST provide the complete, updated list of template identifiers on the target node, rather than merging with or appending to it. There are
three cases when modifying the "apply-templates" annotation:

*  The "apply-templates" annotation is specified and the value is non-
   empty (i.e. a list of templates to apply to the node). The "apply-
   templates" metadata is changed to match the exact value in the request.


*  The "apply-templates" annotation is specified and the value is either empty or
   contains only whitespace. The "apply-templates" metadata is removed and thus no
   templates are applied to the node.

*  The "apply-templates" annotation is not specified. The "apply-templates"
   metadata currently present on the node (if any) is unchanged.

For example, {{update-application-1}} creates two interface entries named "loopback0" and "eth0" and applies template "base-interface" to the interfaces container:

~~~~
<interfaces xmlns="urn:example:interface"
  xmlns:ct="urn:ietf:params:xml:ns:yang:ietf-config-template"
  ct:apply-templates="base-interface">
  <interface>
    <name>loopback0</name>
  </interface>
  <interface>
    <name>eth0</name>
  </interface>
</interfaces>
~~~~
{: #update-application-1 title="An Initial Template Application Example" artwork-align="center"}

A subsequent request in {{update-application-2}} also applies template "ethernet-interface" to the interfaces container:

~~~~
<interfaces xmlns="urn:example:interface"
            ct:apply-templates="ethernet-interface base-interface"/>
~~~~
{: #update-application-2 title="Request to Prepend a Second Template" artwork-align="center"}

After this request, \<running\> is as shown in {{update-application-3}}:

~~~~
<interfaces xmlns="urn:example:interface"
  xmlns:ct="urn:ietf:params:xml:ns:yang:ietf-config-template"
  ct:apply-templates="ethernet-interface base-interface">
  <interface>
    <name>loopback0</name>
  </interface>
  <interface>
    <name>eth0</name>
  </interface>
</interfaces>
~~~~
{: #update-application-3 title="Running Contents After Multiple Template Application" artwork-align="center"}

{{update-application-4}} adds a new interface list entry, and leaves the applied
templates unchanged:

~~~~
<interfaces xmlns="urn:example:interface">
  <interface>
    <name>eth1</name>
  </interface>
</interfaces>
~~~~
{: #update-application-4 title="Request to Add a New Interface Entry" artwork-align="center"}

After this request, \<running\> is as shown in {{update-application-5}}.

~~~~
<interfaces xmlns="urn:example:interface"
  xmlns:ct="urn:ietf:params:xml:ns:yang:ietf-config-template"
  ct:apply-templates="ethernet-interface base-interface">
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
{: #update-application-5 title="Running Contents After Interface Addition" artwork-align="center"}

Finally, this request deletes all the templates, and leaves the list
entries unchanged:

~~~~
<interfaces xmlns="urn:example:interface"
  xmlns:ct="urn:ietf:params:xml:ns:yang:ietf-config-template"
  ct:apply-templates="">
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
{: #update-application-6 title="Request to Clear All Applied Templates" artwork-align="center"}

After this request, \<running\> is as follows:

~~~~
<interfaces xmlns="urn:example:interface">
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
{: #update-application-7 title="Running Contents After Removing Template Applications" artwork-align="center"}

## Overriding Templates {#overriding-temp}

The client may want to to override some configuration in a template
 when it is applied to a particular node in read-write datastores (e.g., \<running\> or \<candidate\>).  The client can
 achieve this by providing the desired value at the corresponding
 level when applying the template.  Configuration explicitly provided
 by the client always takes precedence over the same node defined in
 template.

 A template node can be overriden by having its value changed, but it
 can't be deleted.

 {{override-template}} provides an example of overriding a node in a template, a client may
 configure physically present interfaces "eth0" and "eth1" inheriting
 the template defined in {{base-template}}, but the "mtu" value of "eth1" needs
 to be 9122:

~~~~
<interfaces xmlns="urn:example:interface"
  xmlns:ct="urn:ietf:params:xml:ns:yang:ietf-config-template"
  ct:apply-templates="base-interface">
  <interface>
    <name>eth0</name>
  </interface>
  <interface>
    <name>eth1</name>
    <mtu>9122</mtu>
  </interface>
</interfaces>
~~~~
{: #override-template title="Example of Explicit Configuration Overriding a Template" artwork-align="center"}

 And the above interface configuration renders the expanded
 configuration shown in {{override-template-expansion}}.

~~~~
    <interfaces xmlns="urn:example:interface">
      <interface>
        <name>eth0</name>
        <enabled>true</enabled>
        <mtu>65536</mtu>
        <description>default provisioned interface</description>
      </interface>
      <interface>
        <name>eth1</name>
        <enabled>true</enabled>
        <mtu>9122</mtu>
        <description>default provisioned interface</description>
      </interface>
    </interfaces>
~~~~
{: #override-template-expansion title="Expanded Configuration Result with Overridden MTU" artwork-align="center"}

## Expanding Templates {#expand-templates}

When a configuration template is applied to a node in the data tree,
it acts as if the configuration defined in the template is merged
with the configuration provided explicitly at the corresponding level
in the data tree, with the explicitly provided configuration taking
precedence.

the process of expanding templates to derive \<intended\> is deterministic and depends solely on the contents of \<running\>.
The process rules are as follows:

*  The value of a node in \<intended\> after template expansion is determined
   by using precedence to decide where to take the value from.

*  Non-template configuration always has the highest precedence.

*  When templates are applied to multiple ancestors, the innermost
   ancestor takes precedence.

*  When multiple templates are applied to a particular node, the
   order of application (as indicated by the client when applying the
   templates) determines the precedence within that node.

If a client has knowledge of the complete contents of \<running\> and \<system\>,
it can calculate the exact result of template expansion, independent of the server's operational state.

Whenever the contents of an applied template is updated in \<running\> or \<system\>, the
result of template expansion appears in \<intended\>.

## Deletion of Templates

A configuration template can not be deleted if it is currently actively applied to any data node.
When a client attempts to delete a template definition from read-write datastores (e.g., \<running\> or \<candidate\>) that is in use, the server MUST reject the deletion request with the error-tag value "data-missing", indicating that the template is still in use.

To successfully delete a template, a client MUST first update the target configuration nodes to remove the template identifier from their "apply-templates" metadata attribute (as described in {{apply-templates}}), and then subsequently delete the template definition itself.

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
<CODE BEGINS> file "ietf-config-template@2026-07-03.yang"
{::include ./yang/ietf-config-template.yang}
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

This appendix tracks the status of requirements identified on the
[Template Requirements Issue Tracker([https://github.com/netmod-wg/template-reqs/issues).


R1: [Wherever a template-reference can occur, more than one template-reference can occur (and they are applied)](https://github.com/netmod-wg/template-reqs/issues/1)

  - discussed: strongly in favor
  - status: supported in document (done)

R2: [Templates must be able to reference other templates (hierarchal templates)](https://github.com/netmod-wg/template-reqs/issues/2)

  - discussed: split opinion (needs more discussion)
  - unsure if opposed to idea or to doing it in a first release.
  - status: not supported in document (needs more discussion)

R3: [Templates must work with any YANG module (including augments and deviations)](https://github.com/netmod-wg/template-reqs/issues/3)

  - discussed: strongly in favor
  - status: supported in document (done)

R4: [Template syntax must be validated when defined (not only when used)](https://github.com/netmod-wg/template-reqs/issues/4)

  - discussed: split opinion (needs more discussion)
  - unsure if opposed to idea or to doing it in a first release.
  - status: supported in document (done)

R5: [Wherever a template-reference can occur, it must be possible to delete nodes from the template](https://github.com/netmod-wg/template-reqs/issues/5)

  - discussed: mildly NOT in favor
  - status: not supported in document (done)

R6: [Local-config overrides template-config](https://github.com/netmod-wg/template-reqs/issues/6)

  - discussed: strongly in favor
  - status: supported in document (done)

R7: [Templates are persistent (living templates) modifications to them are automatically applied to all consumers](https://github.com/netmod-wg/template-reqs/issues/7)

  - discussed: strongly in favor
  - status: supported in document (done)

R8: [Support basic programmatic elements in templates](https://github.com/netmod-wg/template-reqs/issues/8)

  - discussed: strongly opposed (but Joe Clarke later said he would've voted in favor)
  - unsure if opposed to idea or to doing it in a first release.
  - status: NOT supported in document (discuss more?)

R9: [It must be possible to constrain which nodes can be template-consumers](https://github.com/netmod-wg/template-reqs/issues/9)

  - discussed: strongly in favor
  - status: NOT supported in document (need to add)

R10: [For living templates, the configuration with both unexpanded and expanded templates is able to be returned](https://github.com/netmod-wg/template-reqs/issues/10)

  - discussed: strongly in favor
  - status: supported in document (done)

R11: [Possibility to reorder some user-ordered list/leaf-list entries defined in a template](https://github.com/netmod-wg/template-reqs/issues/11)

  - discussed: strongly opposed
  - status: not supported in document (done)

R12: [The \<running\> datastore contains the unexpanded template](https://github.com/netmod-wg/template-reqs/issues/12)

  - discussed: strongly in favor
  - status: supported in document (done)

R13: [For NMDA, the \<intended\> datastore returns the expanded template](https://github.com/netmod-wg/template-reqs/issues/13)

  - discussed: strongly in favor
  - status: supported in document (done)

R14: [Off-box template-expansion of \<running\> containing templates must be possible (potentially enabling off-box validation)](https://github.com/netmod-wg/template-reqs/issues/14)

  - discussed: strongly in favor
  - status: supported in document (done)

R15: [For NMDA, the \<intended\> datastore returns the unexpanded templates](https://github.com/netmod-wg/template-reqs/issues/15)

  - discussed: mostly opposed
  - status: supported in document (done)

R16: [Common data nodes](https://github.com/netmod-wg/template-reqs/issues/16)

  - never discussed: this requirement is closed in GitHub
  - status: not supported in document (done)

R17: [For NMDA, The \<operational\> datastore returns unexpanded template config (depends on #15)](https://github.com/netmod-wg/template-reqs/issues/17)

  - discussed: mostly opposed
  - status: not supported in document (done)

R18: [Support limited-regex in template-config for template-consumer application](https://github.com/netmod-wg/template-reqs/issues/18)

  - discussed: strongly in favor
  - status: supported in document (done)

R19: [When multiple templates are applied to a node, a precedence order must be defined (either ascending or descending)](https://github.com/netmod-wg/template-reqs/issues/19)

  - discussed: strongly in favor
  - status: supported in document (done)

R20: [When templates are applied at multiple ancestor nodes, the innermost (closest) template takes precedence.](https://github.com/netmod-wg/template-reqs/issues/20)

  - discussed: mostly in favor
  - status: supported in document (done)

R21: [The solution enables non-nmda servers to return the expanded data](https://github.com/netmod-wg/template-reqs/issues/21)

  - discussed: strongly in favor
  - status: not supported in document (need to add, see {{interact-non-NMDA}})

R22: [Method to exclude templates applied at ancestor nodes](https://github.com/netmod-wg/template-reqs/issues/22)

  - never discussed
  - status: needs to be discussed (will schedule an Interim meeting)

R23: [Clarify the ability to apply a template at the datastore root node '/'](https://github.com/netmod-wg/template-reqs/issues/23)

  - never discussed
  - status: needs to be discussed (will schedule an Interim meeting)

R24: [Metadata annotation to determine which template a node was applied from](https://github.com/netmod-wg/template-reqs/issues/24)

  - never discussed
  - status: needs to be discussed (will schedule an Interim meeting)

R25: [Misaligned module template name](https://github.com/netmod-wg/template-reqs/issues/25)

  - never discussed
  - status: needs to be discussed (will schedule an Interim meeting)

R26: [Seems a typo in this example](https://github.com/netmod-wg/template-reqs/issues/26)

  - never discussed
  - status: needs to be discussed (will schedule an Interim meeting)

R27: [Adding an example for this might help](https://github.com/netmod-wg/template-reqs/issues/27)

  - never discussed
  - status: needs to be discussed (will schedule an Interim meeting)

R28: [Good to have apply-groups-except equivalent feature as in Junos](https://github.com/netmod-wg/template-reqs/issues/28)

  - never discussed
  - status: needs to be discussed (will schedule an Interim meeting)

R29: [Provide the ability to see the expanded view of configuration](https://github.com/netmod-wg/template-reqs/issues/29)

  - never discussed
  - status: needs to be discussed (will schedule an Interim meeting)

R30: [Can the template be applied to a leaf/leaf-list?](https://github.com/netmod-wg/template-reqs/issues/30)

  - never discussed
  - status: needs to be discussed (will schedule an Interim meeting)



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
