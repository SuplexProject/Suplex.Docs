# ISecureObject Overview

The primary interface for Suplex is a hierarchy of ISecureObjects, where each ISecureObject carries a SecurityDescriptor, which, as a structure, describes the security disposition of the object itself.  By analogy, an ISecureObject can be thought of like a directory on a computer - the directory metadata contains a description of who has what kind of access, and how that access was granted (or denied) - either by directly-applied permissions or through inheritance.

At runtime, security information is selected from a Suplex data store and then used to populate the SecurityDescriptor of an ISecureObject.  The SecurityDescriptor carries two lists: the DiscretionaryAccessControlList (DACL), which is a list of permissions, and the SystemAccessControlList (SACL), which is a list of audit entries.  The items in the ACLs are called AccessControlEntries (ACEs), where each ACE specifies who has what kind of access, and whether that access is audited.

## ISecureObject

The properties supporting an ISecureObject are just enough to identify it in a datastore and represent its hierarchy.

|Field|Type|Required|Description
|-|-|-|-
|UId|Guid|Yes|Primary key in the datastore, a GUID to support replication amongst stores.
|UniqueName|string|Yes|Human-identifiable name for an object, should be unique within a hierarchy.
|ParentUId|Nullable Guid|No|The GUID of the current ISecureObject's Parent, if there is a Parent.
|Parent|ISecureObject|No|The current ISecureObject's Parent, if there is a Parent.
|Children|List|No|A collection of the current object's child-ISecureObjects, if there are any.
|Security|ISecurityDescriptor|Yes|The SecurityDescriptor for the ISecureObject contains a description of the security disposition.

## ISecurityDescriptor

|Field|Type|Required|Description
|-|-|-|-
|DaclAllowInherit|bool|Yes|Specifies if the Dacl can inherit permission Aces from parent-ISecureObject-Dacls.
|SaclAllowInherit|bool|Yes|Specifies if the Sacl can inherit audit Aces from parent-ISecureObject-Sacls.
|SaclAuditTypeFilter|AuditType|Yes|Specifies the granularity of Audit messages.  See below for detail.
|Dacl|IDiscretionaryAcl|Yes|A list of permission Aces for the current ISecureObject.  At runtime, inheritable permission Aces are evaluated as part of the current Dacl.
|Sacl|ISystemAcl|Yes|A list of audit Aces for the current ISecureObject.  At runtime, inheritable audit Aces are evaluated as part of the current Sacl.
|Results|SecurityResults|Yes|System-generated list of resultant security, calculated post-eval.

```c#
[Flags]
public enum AuditType
{
    SuccessAudit = 1,
    FailureAudit = 2,
    Information = 4,
    Warning = 8,
    Error = 16,
    Detail = 32
}
```

## AccessControlLists (Acls)

### DiscretionaryAcl (Dacl (Permissions, Rights))

The Discretionary Access Control List is a collection of IAccessControlEntry (Aces).  A Dacl specifies whether specific permissions are granted/denied to an object.

### SystemAcl (Sacl)

The System Access Control List is a collection of IAccessControlEntryAudit (AuditAces).  A Sacl specifies what permissions (as specified in the Dacl) will generate audit records.

## IAccessControlEntry\<T> (Ace)

|Field|Type|Required|Description
|-|-|-|-
|UId|Guid|Yes|Primary key in the datastore, a GUID to support replication amongst stores.
|Allowed|bool|Yes|A value of `true` grants access to a permission, and a value of `false` explicitly denies access to a permission.  **Important**: An explicit deny __always__ overrides a grant.
|Inheritable|bool|Yes|Specifies whether the current Ace will propagate to child Dacls.
|InheritedFrom|Nullable Guid|No|System-populated at runtime; the UId of the original Ace from which the current entry was created.
|TrusteeUId|Nullable Guid|No*|The UId of the User or Group to which the Ace is assigned.  When the containing Acl is evaluated, any Aces present will be included in the resultant security calculation.  Strictly speaking, Trustee-free Aces could be inserted into an Acl at runtime and evaluated safely.
|Right|T|Yes|T can be any Enum with a Flags attribute, such that individual rights can be combined for a single permission entry.  Built-in Rights enums are shown below.

```c#
[Flags]
public enum UIRight
{
    FullControl = 7,
    Operate = 4,
    Enabled = 2,
    Visible = 1
}

[Flags]
public enum RecordRight
{
    FullControl = 31,
    Delete = 16,
    Update = 8,
    Insert = 4,
    Select = 2,
    List = 1
}

[Flags]
public enum FileSystemRight
{
    FullControl = 511,
    Execute = 256,
    Delete = 128,
    Write = 64,
    Create = 32,
    Read = 16,
    List = 8,
    ChangePermissions = 4,
    ReadPermissions = 2,
    TakeOwnership = 1
}

[Flags]
public enum SynchronizationRight
{
    TwoWay = 7,
    Upload = 5,
    Download = 3,
    OneWay = 1
}
```

## IAccessControlEntryAudit (AuditAce)

An IAccessControlEntryAudit ace inherits all the properties from IAccessControlEntry\<T>, but changes the behavior of `Allowed` and additionally implements `Denied`, as described below.

|Field|Type|Required|Description
|-|-|-|-
|Allowed|bool|Yes|Specifies whether to audit grants for a permission.
|Denied|bool|Yes|Specifies whether to audit explicit denies for a permission.

## IAccessControlEntryConverters\<TSource, TTarget> (Permission Converters)

An IAccessControlEntryConverter translates the resultant security for any given Ace/Right into a new Ace/Right of any other Ace type.  The new Ace's `Allowed` property is based on the source-Ace security result, and the new Ace can be configured with standard `Inheritable` settings.

|Field|Type|Required|Description
|-|-|-|-
|TSource|Right Type|Yes|Specifies the Right type from which to take the resultant Allowed value.
|Source Right|Right Value|Yes|Specifies the Right value from which to take the resultant Allowed value.
|TTarget|Right Type|Yes|Specifies the Right type (T) to be used when creating a new IAccessControlEntry\<T>.
|Target Right|Right Value|Yes|Specifies the Right value (of T) for the new Ace.
|Inheritable|bool|Yes|Specifies whether the new Ace will propagate to child Dacls.

## Results (SecurityResults)

A dictionary of of SecurityResult entries, set by the system at runtime when a SecurityDescriptor is evaluated.  One SecurityResult object will exist in the dictionary for each Right-type, where the SecurityResult object is defined as:

|Field|Type|Required|Description
|-|-|-|-
|RightName|string|Yes|The name of the permission.
|AccessAllowed|bool|Yes|Specifies if access was granted or denied.
|AuditSuccess|bool|Yes|Specifies if the permission will be audit for AccessAllowed = true.
|AuditFailure|bool|Yes|Specifies if the permission will be audit for AccessAllowed = false.