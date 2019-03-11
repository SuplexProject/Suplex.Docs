# Best Practice for Suplex RBAC Design

## Why Suplex?

Role Based Access Control (RBAC) is neither new nor exciting, but solid, reliable security design is critical for almost all applications.  Suplex evolves RBAC design by _securing by concept_ and pushing the application security implementation to config, thereby introducing extensibility patterns.

I wrote Suplex because I got tired of copy/pasting code into similar (but always a _little_ different) proprietary RBAC models for each new application, and I also wanted to make security administration easier over the lifetime of an application.

## Traditional RBAC v Securing by Concept

#### Traditional RBAC

A traditional RBAC is is typically *Users/Groups => Role => Right*, where Rights are defined in application-specific ideas and implemented in matching code blocks.  The drawback to this approach is one must understand all possible security profiles before coding, or the Rights must be sufficiently granular to accommodate unforseen and anomalous profiles.  As developers, we'll probably choose granularity, but, the resultant RBAC Rights list can end up being quite large, which may confuse consumers or just be a lot of work to maintain.  We could consolidate, but that, of course, defeats the granularity.  Another general problem is alignment of RBAC Rights definitions to code - not every code action neatly matches discreet Rights, thus hampering optimally granulating Rights.

In the example diagram below, the initial RBAC is defined with Rights 1, 2, and 3, mapped to corresponding code actions, and there's only one bit of shared-Rights-code - so only slightly unhappy.  The problem doesn't come until later, when an anomalous Role request comes about - Role4 - which requires defining an unforseen Right4.

- **Option 1**: Grant users access to Roles 1, 2, and 3.  That works but that provides greater access than desired.  When the request arrives, this is often the chosen solution.
- **Option 2**: Program a new Role(4) that properly defines Right4.  This requires code maintenance, regression testing, documentation, and more.
- **Both options** make us sad, and we can predict it's only going to get worse, which makes us sadder still.

![Traditional RBAC](../img/trad_rbac.png "Traditional RBAC")
