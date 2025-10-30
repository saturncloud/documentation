# Real-Time Collaboration in Jupyter

{{% alert title="Git is Recommended for Collaboration" %}}
For most team workflows, collaborating through Git is more productive and reliable than real-time collaboration. Real-time collaboration should be used for specific use cases like pair programming, live demos, or interactive teaching sessions.
{{% /alert %}}

JupyterLab supports real-time collaboration, allowing multiple users to work simultaneously in the same notebooks and files. This feature can be useful for pair programming, live demonstrations, or training sessions where multiple people need to view and edit the same notebook at the same time.

## Prerequisites

Before setting up real-time collaboration, you need to:

- Have a Jupyter server resource created (see [Jupyter Servers](<docs/user-guide/how-to/resources/jupyter-servers.md>))
- Install the Jupyter collaboration extension
- Configure access permissions for collaborators

## Install Jupyter Collaboration Extension

To enable real-time collaboration, you need to install the `jupyter-collaboration` package into the environment that runs JupyterLab itself (not the data science environment). This must be done using the **Start Script**.

{{% alert title="Important" %}}
The **Extra Packages** section installs packages into your data science environment, but `jupyter-collaboration` needs to be installed into the JupyterLab environment at `/opt/saturncloud`. Therefore, you must use the start script method below.
{{% /alert %}}

Add the following to your resource's **Advanced Settings > Start Script**:

```bash
/opt/saturncloud/bin/pip install jupyter-collaboration
```

You can add this when creating a new resource, or by editing an existing resource:

1. Navigate to your Jupyter server resource page
2. Click on the resource name to edit settings
3. Expand **Advanced Settings** at the bottom of the form
4. In the **Start Script (Bash)** section, add the pip install command above
5. Click **Update** to save your changes
6. Restart your resource for the changes to take effect

Using `/opt/saturncloud/bin/pip` ensures the package is installed directly into the JupyterLab environment where it can be loaded.

## Configure Access Permissions

Once the collaboration extension is installed, you need to grant other users access to your Jupyter server. There are two approaches:

### Option 1: Add Viewers to Your Jupyter Server

If you own the Jupyter server, you can add specific users or groups as viewers:

1. Navigate to your Jupyter server resource page
2. Look for the **Viewers** section in the resource details
3. Click **Add Viewer**
4. Select the users or groups you want to grant access to
5. Save your changes

![Adding viewers to a resource](/images/docs/viewers.webp "doc-image")

{{% alert title="Important: Viewer Limitations" %}}
Viewers cannot start, stop, or delete the Jupyter server resource. As the owner, you must:
1. Start the Jupyter server yourself
2. Copy the JupyterLab URL from your browser
3. Share this URL directly with the viewers

The JupyterLab URL does not change, so viewers can save this URL and reuse it for future sessions. However, viewers will only be able to access the JupyterLab session when the server is running.
{{% /alert %}}

### Option 2: Use a Group-Owned Jupyter Server

Instead of individually adding viewers, you can create a Jupyter server owned by a group. All members of the group will automatically have access to the server:

1. When creating a new Jupyter server, set the **Owner** field to a group instead of your user account
2. All group members will be able to start, stop, and access the server
3. Any group member can collaborate in real-time when the server is running

For more information about groups, see the [Groups documentation](/docs).

{{% alert title="Note" %}}
The owner of a Jupyter server (user or group) cannot be changed after creation. Plan your ownership structure accordingly.
{{% /alert %}}

## Using Real-Time Collaboration

Once the extension is installed and access is configured, collaborators can work together:

1. The resource owner starts the Jupyter server
2. Collaborators navigate to the same resource page in Saturn Cloud
3. All users can open the same notebooks and files
4. Changes made by any user appear in real-time for all collaborators
5. Each user's cursor and selections are shown with different colors

### Collaboration Features

- **Simultaneous editing**: Multiple users can edit the same notebook cells
- **Live cursors**: See where other users are working with colored cursors
- **Automatic sync**: Changes are synchronized automatically across all users
- **Shared execution**: Code execution happens on the shared resource

### Best Practices

- **Communication**: Use a separate communication tool (Slack, Zoom, etc.) to coordinate with collaborators
- **Resource size**: Ensure the Jupyter server has sufficient resources for multiple users
- **Save frequently**: Although changes sync in real-time, regular saves are recommended
- **Git integration**: Combine real-time collaboration with Git for version control
- **Clear ownership**: Establish clear guidelines for who can start/stop group-owned resources

## Limitations and Considerations

- **Performance**: Real-time collaboration adds overhead. Use appropriately sized instances for your team
- **Network dependency**: Requires stable internet connections for all collaborators
- **Merge conflicts**: Simultaneous edits to the same cell can be confusing. Coordinate with collaborators
- **Not a replacement for Git**: Real-time collaboration doesn't provide version control, code review, or persistent history

## When to Use Real-Time Collaboration

Real-time collaboration works best for:

- Pair programming sessions
- Live training or demos
- Debugging sessions with team members
- Interactive code reviews
- Short-term collaborative analysis

For asynchronous collaboration, code reviews, and production workflows, use Git and [resource recipes](/docs) instead.

