# What is Suplex?

Suplex.Security is an application security and RBAC abstraction Layer, which implements a hierarchical DACL model and common role/task RBAC model. Suplex is suitable for use in any application/API.

## Philosophy

The intent of Suplex is:

1. To support a flexible, adaptable, pluggable RBAC that's abstracted from core application design,
2. To avoid tightly-coupling the security Role implementation to application features or modules, and thus:
3. Avoid "role bleed," where existing Role-implementation to app-feature encompasses too great a scope, and over time requires code maintenance to narrow the scope, where otherwise there is insufficient granularity in Roles manifested at the point of consumption.

## Design Overview

The primary interface for Suplex is a hierarchy of ISecureObjects, where each ISecureObject carries a SecurityDescriptor, which, as a structure, describes the security disposition of the object itself.  By analogy, an ISecureObject can be thought of like a directory on a computer - the directory metadata contains a description of who has what kind of access, and how that access was granted (or denied) - either by directly-applied permissions or through inheritance.

At runtime, security information is selected from a Suplex data store and then used to populate the SecurityDescriptor of an ISecureObject.  The SecurityDescriptor carries two lists: the DiscretionaryAccessControlList (DACL), which is a list of permissions, and the SystemAccessControlList (SACL), which is a list of audit entries.  The items in the ACLs are called AccessControlEntries (ACEs), where each ACE specifies who has what kind of access, and whether that access is audited.

## Anatomy of ISecureObject

