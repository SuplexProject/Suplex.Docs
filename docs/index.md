# What is Suplex?

Suplex.Security is an application security and RBAC abstraction Layer, which implements a hierarchical DACL model and common role/task RBAC model. Suplex is suitable for use in any application/API.

## Philosophy

The intent of Suplex is:

1. To support a flexible, adaptable, pluggable RBAC that's abstracted from core application design,
2. To avoid tightly-coupling the security Role implementation to application features or modules, and thus:
3. Avoid "role bleed," where existing Role-implementation to app-feature encompasses too great a scope, and over time requires code maintenance to narrow the scope, where otherwise there is insufficient granularity in Roles manifested at the point of consumption.

# QuickStart

Wanna get started without a lot of pain?  Me too.  Given the philosophical statement above, you won't need much code to use Suplex in your application.  This quick start guide assumes a working knowledge of Visual Studio, C#, and nuget.

## Get the Admin UI

Download the latest release of <a href="https://github.com/SuplexProject/Suplex.UI.Wpf/releases" target="_blank">Suplex.UI.Wpf</a> from GitHub.  To get started, extract the zip and run the exe.

### 1. Create some Security Principals (Users and Groups)

### 2. Create some Secure Objects


