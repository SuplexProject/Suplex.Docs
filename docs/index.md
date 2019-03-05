# What is Suplex?

Suplex.Security is an application security and RBAC abstraction Layer, which implements a hierarchical DACL model and common role/task RBAC model. Suplex is suitable for use in any application/API.

### Philosophy

The intent of Suplex is:

1. To support a flexible, adaptable, pluggable RBAC that's abstracted from core application design,
2. To avoid tightly-coupling the security Role implementation to application features or modules, and thus:
3. Avoid "role bleed," where existing Role-implementation to app-feature encompasses too great a scope, and over time requires code maintenance to narrow the scope, where otherwise there is insufficient granularity in Roles manifested at the point of consumption.

## QuickStart

Wanna get started without a lot of pain?  Me too.  Given the philosophical statement above, you won't need much code to use Suplex in your application.  This quick start guide assumes a working knowledge of Visual Studio, C#, and NuGet.

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

- Once you're finished creating Security Principals and Secure Objects, save your work to a file - click the _Save Suplex File Store_ icon from the toolbar.

<!-- 2980B9 -->
- <details style="cursor: hand;"><summary><font style="background-color:#BBD7E9;">Click here</font> to expand this section for animated samples of editing a Suplex FileStore.</summary>
<p align="center">
<iframe width="560" height="315" src="https://www.youtube.com/embed/FimK9o6e7l0" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</p></details>

#### 2. Use the Suplex FileStore in an App

This example uses a simple WinForms app to demonstrate Suplex integration.  The highlights are:

- NuGet references to <a href="https://www.nuget.org/packages/Suplex.Security.Core/" target="_blank">Suplex.Security.Core</a> and <a href="https://www.nuget.org/packages/Suplex.Security.FileSystemDal/" target="_blank">Suplex.Security.FileSystemDal</a>.

- A FileWatcher to detect updates to the FileStore and dynamically reload security information.

- A combobox to simulate switching security context.

- Full source here: <a href="https://github.com/SuplexProject/Suplex.Sample" target="_blank">Suplex.Sample</a>

Most of the code in the sample app is setup-oriented. not functionally relevant to Suplex itself.  Below are the key functions to instantiating the FileSystemDal and selecting security at runtime.  The critical code elements are:

```c#
//Load the Suplex FileStore from disk
_suplexDal = FileSystemDal.LoadFromYamlFile( filestorePath );

//Eval the security for an object at runtime.
//Note: The return value may be null if the object is not found, the user is disabled, or for other reasons.
//      Be sure to null-check usage.
SecureObject secureObject =
    (SecureObject)_suplexDal.EvalSecureObjectSecurity( "frmMain", ((User)cmbUsers.SelectedItem).Name );

//Assess 'AccessAllowed' (bool) for the object or a descendant (child) object
secureObject?.Security.Results.GetByTypeRight( UIRight.Visible ).AccessAllowed;
secureObject?.FindChild<SecureObject>( "txtId" ).Security.Results.GetByTypeRight( UIRight.Visible ).AccessAllowed;
```

In context, here's the same code as applied within the relevant methods:
```c#
public partial class MainDlg : Form
{
    FileSystemDal _suplexDal = new FileSystemDal();

    /// <summary>
    /// Loads the specified file into a Suplex FileSystemDal and refreshes the dialog
    /// </summary>
    /// <param name="filestorePath"></param>
    void RefreshSuplex(string filestorePath)
    {
        _suplexDal = FileSystemDal.LoadFromYamlFile( filestorePath );
        this.UiThreadHelper( () => cmbUsers.DataSource = new BindingSource( _suplexDal.Store.Users.OrderBy( u => u.Name ).ToList(), null ).DataSource );
    }

    /// <summary>
    /// Simulates changing security context.  Normally, this would occur at the time of application/method invocation.
    /// </summary>
    private void cmbUsers_SelectedIndexChanged(object sender, EventArgs e)
    {
        //Evaluate the security information, starting from the top-most control
        SecureObject secureObject =
            (SecureObject)_suplexDal.EvalSecureObjectSecurity( "frmMain", ((User)cmbUsers.SelectedItem).Name );

        if( chkApplyRecursive.Checked )
            ApplyRecursive( secureObject );
        else
            ApplyDirect( secureObject );
    }

    /// <summary>
    /// Brute-force permissioning - direct lookup of results with "known" translation of non-UI rights
    /// </summary>
    /// <param name="secureObject">A reference to the resolved/evaluated security object.</param>
    void ApplyDirect(SecureObject secureObject)
    {
        frmMain.Visible = secureObject?.Security.Results.GetByTypeRight( UIRight.Visible ).AccessAllowed ?? false;
        txtId.Visible = secureObject?.FindChild<SecureObject>( "txtId" ).Security.Results.GetByTypeRight( UIRight.Visible ).AccessAllowed ?? false;
        btnCreate.Enabled = secureObject?.FindChild<SecureObject>( "btnCreate" ).Security.Results.GetByTypeRight( RecordRight.Insert ).AccessAllowed ?? false;
    }

    /// <summary>
    /// Recursive, discovery-based security application; see UIExtensions->ApplySecurity
    /// </summary>
    /// <param name="secureObject">A reference to the resolved/evaluated security object.</param>
    void ApplyRecursive(SecureObject secureObject)
    {
        frmMain.ApplySecurity( secureObject );
    }
}

public static class UIExtensions
{
    public static void ApplySecurity(this Control control, ISecureObject secureObject)
    {
        if( secureObject == null )
        {
            control.Visible = false;
            return;
        }

        ISecureObject found = secureObject.UniqueName.Equals( control.Name, StringComparison.OrdinalIgnoreCase ) ?
            secureObject : secureObject.FindChild<ISecureObject>( control.Name );
        if( found != null && found.Security.Results.ContainsRightType( typeof( UIRight ) ) )
        {
            control.Visible = found.Security.Results.GetByTypeRight( UIRight.Visible ).AccessAllowed;
            control.Enabled = found.Security.Results.GetByTypeRight( UIRight.Enabled ).AccessAllowed;
            if( control is TextBox )
                ((TextBox)control).ReadOnly = !found.Security.Results.GetByTypeRight( UIRight.Operate ).AccessAllowed;
        }

        if( control.HasChildren )
            foreach( Control child in control.Controls )
                child.ApplySecurity( secureObject );
    }
}
```