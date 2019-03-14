# SecurityPrincipal Overview

SecurityPrincipals are Users and Groups.  In an RBAC, Roles exist to assign Rights - a Suplex Role is just a Group that is dedicated to Rights assignment.  Role membership - other Users and Groups - need to be defined in Suplex, but they may be sourced from a another provider, such as an LDAP store, for general management.  For group membership management, Suplex differentiates between internal (local) and external groups, where local Groups allow membership edits and external groups do not.  User objects only need to be defined in Suplex if assigning Users directly to Group membership.  If sourcing another provider for external groups, the external groups will be members of the Role-groups, and thus User objects may not be required.

## ISecurityPrincipal

ISecurityPrincipal is the base interface for Users and Groups.

#### Properties

|Field/Method|Type|Required|Description
|-|-|-|-
|UId|Guid|Yes|Primary key in the datastore, a GUID to support replication amongst stores.
|Name|string|Yes|Gets or sets the account name.
|Description|string|No|Gets or sets the account description.
|IsLocal|bool|Yes|Gets or sets the internal/external status of the account
|IsBuiltIn|bool|Yes|Gets or sets an indicator for "required" accounts
|IsEnabled|bool|Yes|Gets or sets the enabled status of the account
|IsValid|bool|Yes|(_deprecated_) Gets or sets an indicator for properly initialized accounts
|IsUser|bool|Yes|Gets a value indicating if the current object is a User or a Group

## User

A User object inherits the base definition of ISecurityPrincipal and implements the following additional properties.

#### Properties

|Field/Method|Type|Required|Description
|-|-|-|-
|IsAnonymous|bool|Yes|Gets or sets whether the User is resolved against the Suplex store.  This is only valid in applications that allow anonymous authentication paradigms.
|IsUser|bool|Yes|Always returns `true`

## Group

ISecurityPrincipal is the base interface for Users and Groups.

#### Properties

|Field/Method|Type|Required|Description
|-|-|-|-
|Mask|byte[]|No|A unique value used in Row-Level Security (RLS) implementations
|IsUser|bool|Yes|Always returns `false`
|Groups|collection|No|A list of child Group objects; nested group membership
