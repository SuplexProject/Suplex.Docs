# What is Suplex?

Suplex.Security is an application security and RBAC abstraction Layer, which implements a hierarchical DACL model and common role/task RBAC model. Suplex is suitable for use in any application/API.

## Philosophy

The intent of Suplex is:

1. To support a flexible, adaptable, pluggable RBAC that's abstracted from core application design,
2. To avoid tightly-coupling the security Role implementation to application features or modules, and thus:
3. Avoid "role bleed," where existing Role-implementation to app-feature encompasses too great a scope, and over time requires code maintenance to narrow the scope, where otherwise there is insufficient granularity in Roles manifested at the point of consumption.

## QuickStart

Wanna get started without a lot of pain?  Me too.  Given the philosophical statement above, you won't need much code to use Suplex in your application.  This quick start guide assumes a working knowledge of Visual Studio, C#, and nuget.

### Get the Admin UI

Download the latest release of <a href="https://github.com/SuplexProject/Suplex.UI.Wpf/releases" target="_blank">Suplex.UI.Wpf</a> from GitHub.  To get started, extract the zip and run the exe.

#### 1. Create some Security Principals and Secure Objects

- From the toolbar, choose _Security Principals_, then, from the left panel, choose _New > New User_.

    - Complete the properties in the right panel, then click Save.  Create a few Users.
    - Choose _New > New Group_, and create a few Groups in a similar fashion to Users.

        - For each new Group, choose the _Local (Suplex)_ option and add some Users in the bottom _Members_ list.

- Now choose _Secure Objects_ from the toolbar and then _New > New Root_ from the left panel.

    - Complete the UniqueName field, where the Secure Object's UniqueName is the key field you'll use at runtime to identify the security entries in the Suplex store.
    - In the Permissions grid, choose _New Permission > UIRight_. Select a Group and select the desired permission settings.

- <details style="cursor: hand;"><summary>Click here to expand this section for animated eamples of editing a Suplex FileStore.</summary><p align="center"><img alt="Synapse Concept" src="img/SuplexUI.gif" /></p><p align="center"><iframe width="640" height="360" src="https://www.youtube.com/embed/0ZeTzXGVWeg" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></p></details>
