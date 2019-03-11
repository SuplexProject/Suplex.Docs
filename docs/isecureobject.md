# ISecureObject Overview

The primary interface for Suplex is a hierarchy of ISecureObjects, where each ISecureObject carries a SecurityDescriptor, which, as a structure, describes the security disposition of the object itself.  By analogy, an ISecureObject can be thought of like a directory on a computer - the directory metadata contains a description of who has what kind of access, and how that access was granted (or denied) - either by directly-applied permissions or through inheritance.

At runtime, security information is selected from a Suplex data store and then used to populate the SecurityDescriptor of an ISecureObject.  The SecurityDescriptor carries two lists: the DiscretionaryAccessControlList (Dacl), which is a list of permissions, and the SystemAccessControlList (Sacl), which is a list of audit entries.  The items in the ACLs are called AccessControlEntries (Aces), where each Ace specifies who has what kind of access, and whether that access is audited.

## ISecureObject

Suplex ships with a default implementation for ISecureObject, `SecureObject`, but implementing the interface on your own classes is designed to be easy - all the methods of ISecureObject are implemented as extension methods, so you only need to implement the properties.  In doing so, implement Parent/Children as strongly-typed to your class and use the default ISecurityDescriptor implementation, `SecurityDescriptor`.

#### Properties

The properties supporting an ISecureObject are just enough to identify it in a datastore and represent its hierarchy.

|Field/Method|Type|Required|Description
|-|-|-|-
|UId|Guid|Yes|Primary key in the datastore, a GUID to support replication amongst stores.
|UniqueName|string|Yes|Human-identifiable name for an object, should be unique within a hierarchy.
|ParentUId|Nullable Guid|No|The GUID of the current ISecureObject's Parent, if there is a Parent.
|Parent|ISecureObject|No|The current ISecureObject's Parent, if there is a Parent.
|Children|List|No|A collection of the current object's child-ISecureObjects, if there are any.
|Security|ISecurityDescriptor|Yes|The SecurityDescriptor for the ISecureObject contains a description of the security disposition.

#### Key Methods

|Method|Return Value|Description
|-|-|-
|EvalSecurity()|SecurityResults|Recursively calculates the security results for the current object and all descendants.
|FindChild<*T*>( `uniqueName` )|*T*, where T : ISecureObject| Recursively searches the Children hierarchy for a matching: `child.UniqueName.Equals( uniqueName, StringComparison.OrdinalIgnoreCase ) )`


### Example Code
```c#
//Calculate SecurityResults for the object hierarchy
SecurityResults results = secureObject.EvalSecurity();

//Find a child
SecureObject childObject = secureObject.FindChild<SecureObject>( "uniqueName" );
```


## ISecurityDescriptor

|Field|Type|Required|Description
|-|-|-|-
|DaclAllowInherit|bool|Yes|Specifies if the Dacl can inherit permission Aces from parent-ISecureObject-Dacls.  Default is **true**.
|SaclAllowInherit|bool|Yes|Specifies if the Sacl can inherit audit Aces from parent-ISecureObject-Sacls.  Default is **true**.
|SaclAuditTypeFilter|AuditType|Yes|Specifies the granularity of Audit messages.  See below for detail.  Default is  **SuccessAudit \| FailureAudit \| Information \| Warning \| Error**
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

### Example Code
```c#
//create a new SecureObject
SecureObject secureObject = new SecureObject() { UniqueName = "uniqueName" };
//block inheritance of Dacl Aces from parent objects
secureObject.Security.DaclAllowInherit = false;
//populate the Dacl
secureObject.Security.Dacl = new DiscretionaryAcl
{
    new AccessControlEntry<FileSystemRight>
    {
        Allowed = true,
        Right = FileSystemRight.FullControl
    },
    //this is an explicit-Deny Ace, and will not be inherited to child Dacls
    new AccessControlEntry<FileSystemRight>
    {
        //explicit Deny, overrides Execute, List from FullControl above
        Allowed = false,
        Right = FileSystemRight.Execute | FileSystemRight.List,
        //will not be inherited to child Dacls
        Inheritable = false
    },
    new AccessControlEntry<UIRight>
    {
        Right = UIRight.Operate | UIRight.Visible
    }
};
```

## AccessControlLists (Acls)

### DiscretionaryAcl (Dacl (Permissions, Rights))

The Discretionary Access Control List is a collection of IAccessControlEntry (Aces).  A Dacl specifies whether specific permissions are granted/denied to an object.

### SystemAcl (Sacl)

The System Access Control List is a collection of IAccessControlEntryAudit (AuditAces).  A Sacl specifies what permissions (as specified in the Dacl) will generate audit records.

## IAccessControlEntry<*T*> (Ace)

|Field|Type|Required|Description
|-|-|-|-
|UId|Guid|Yes|Primary key in the datastore, a GUID to support replication amongst stores.
|Allowed|bool|Yes|A value of `true` grants access to a permission, and a value of `false` explicitly denies access to a permission.  **Important**: An explicit deny __always__ overrides a grant.
|Inheritable|bool|Yes|Specifies whether the current Ace will propagate to child Dacls.
|InheritedFrom|Nullable Guid|No|System-populated at runtime; the UId of the original Ace from which the current entry was created.
|TrusteeUId|Nullable Guid|No*|The UId of the User or Group to which the Ace is assigned.  When the containing Acl is evaluated, any Aces present will be included in the resultant security calculation.  Strictly speaking, Trustee-free Aces could be inserted into an Acl at runtime and evaluated safely.
|Right|T|Yes|T can be any Enum with a Flags attribute, such that individual rights can be combined for a single permission entry.  Built-in Rights enums are shown below.

### Example Code
```c#
//create a new Ace
//this is an explicit-Deny Ace, and will not be inherited to child Dacls
IAccessControlEntry<FileSystemRight>
    new AccessControlEntry<FileSystemRight>
    {
        //explicit Deny, overrides Execute, List from FullControl above
        Allowed = false,
        Right = FileSystemRight.Execute | FileSystemRight.List,
        //will not be inherited to child Dacls
        Inheritable = false 
    };
```

#### Built-in Right-types

Extend Suplex easily by providing your own enums with a Flags attribute.


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

An IAccessControlEntryAudit ace inherits all the properties from IAccessControlEntry\<T>, but changes the behavior of `Allowed` and additionally implements `Denied`, as described below.  For both Allowed and Denied, audits apply whether Aces are direct or inherited, and for Denied, whether implicit or explicit.

|Field|Type|Required|Description
|-|-|-|-
|Allowed|bool|Yes|Specifies whether to audit grants for a permission.
|Denied|bool|Yes|Specifies whether to audit explicit denies for a permission.

### Example Code
```c#
//create a new AuditAce
//this is an explicit-Deny Ace, and will not be inherited to child Dacls
IAccessControlEntryAudit<FileSystemRight>
    new AccessControlEntryAudit<FileSystemRight>
    {
        //will audit if Execute, List are allowed
        Allowed = true,
        //will audit if Execute, List are denied (implicitly or explicitly)
        Denied = true,
        Right = FileSystemRight.Execute | FileSystemRight.List,
        //will not be inherited to child Dacls
        Inheritable = false
    };
```

## IAccessControlEntryConverters<*TSource, TTarget*> (Permission Converters)

An IAccessControlEntryConverter translates the resultant security for any given Ace/Right into a new Ace/Right of any other Ace type.  The new Ace's `Allowed` property is based on the source-Ace security result, and the new Ace can be configured with standard `Inheritable` settings.

|Field|Type|Required|Description
|-|-|-|-
|TSource|Right Type|Yes|Specifies the Right type from which to take the resultant Allowed value.
|SourceRight|TSource|Yes|Specifies the Right value from which to take the resultant Allowed value.
|TTarget|Right Type|Yes|Specifies the Right type (T) to be used when creating a new IAccessControlEntry\<T>.
|TargetRight|TTarget|Yes|Specifies the Right value (of T) for the new Ace.
|Inheritable|bool|Yes|Specifies whether the new Ace will propagate to child Dacls. Default is **true**.
|UId|Guid|No|Unique Id for identifying the record in a datastore.

#### Example Code
```c#
//Creates an AceConverter to convert RecordRight.Insert -> UIRight.Enabled,
//  where the resultant AccessAllowed value for RecordRight.Insert becomes
//  the Allowed value for UIRight.Enabled.
IAccessControlEntryConverter converter =
    new AccessControlEntryConverter<RecordRight, UIRight>
{
    SourceRight = RecordRight.Insert,
    TargetRight = UIRight.Enabled,
    Inheritable = true
};
```

#### More Information
In effect, AceConverters create new Aces at runtime and inject them into the Dacl.  From the example code above, a new Ace with Right => UIRight.Enabled equal to the resultant AccessAllowed value for RecordRight.Insert from the current ISecureObject will be created, inserted, and subsequently evaluated.

```c#
this.Security.Dacl.Add(
    new AccessControlEntry<UIRight>
    {
        Allowed = this.Security.Results.GetByTypeRight( RecordRight.Insert ).AccessAllowed,
        Right = UIRight.Enabled
    }
);
```

## Results (SecurityResults)

A dictionary of of SecurityResult entries, set by the system at runtime when a SecurityDescriptor is evaluated.  One SecurityResult object will exist in the dictionary for each Right-type, where the SecurityResult object is defined as:

|Field|Type|Required|Description
|-|-|-|-
|RightName|string|Yes|The name of the permission.
|AccessAllowed|bool|Yes|Specifies if access was granted or denied.
|AuditSuccess|bool|Yes|Specifies if the permission will be audited for AccessAllowed = true.
|AuditFailure|bool|Yes|Specifies if the permission will be audited for AccessAllowed = false.

#### Example Code
```c#
//Calculate SecurityResults for the object hierarchy
secureObject.EvalSecurity();

//Assess 'AccessAllowed' (bool) for the object
secureObject.Security.Results.GetByTypeRight( UIRight.Visible ).AccessAllowed;

//Assess 'AccessAllowed' for a descendant (child) object
secureObject.FindChild<SecureObject>( "uniqueName" ).
    Security.Results.GetByTypeRight( RecordRight.List ).AccessAllowed;
```