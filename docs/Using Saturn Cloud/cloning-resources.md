# Clone Resources

If you have a resource that you like using there may be situations where you want to copy it:

* You may want multiple resources to concurrently test your code with different types of hardware, say one with a small CPU and one with a massive CPU.
* You may want to have code interactively in a Jupyter server, then make a job resource to have that code run on a schedule.
* If you're using an enterprise account, you may want to share your environment with another user in your organization.

In Saturn Cloud _cloning_ a resource can accomplish all of these tasks. Cloning makes a new, second resource from an existing one. There are two caveats to cloning:

1. Cloning creates an entirely separate resource. Once you clone a resource, any change you make to the original resource won't be propagated to the new resource. After cloning the two resources are entirely disconnected.
2. Cloning copies most, but not all, of the attributes of a resource to the new one. In particular:
  * Anything stored on the hard drive of the resource won't be copied. _This includes any code you may have written in a Jupyter server and not shared to a git repository._ If you would like to share code when you clone a resource, make sure that code is stored in a [git repository](<docs/Using Saturn Cloud/gitrepo.md>) connected to the resource.
  * If a different user is cloning the resource, credentials will not be cloned.

To clone a resource, click the **Clone Existing Resource** card from the resources page:

![Clone Existing Resource](/images/docs/clone-resource.png "doc-image")

On that page you'll be asked:

* **Resource Type** - the type of resource to create (Jupyter server, RStudio server, Deployment, or Job). This type does not have to be the same as what you're cloning. You may want to, for example, take a Jupyter server and clone it into a job if you want to rerun the task on a schedule
* **Clone From** - The resource to clone. This list will show all your existing resources, and if you're using an Enterprise plan it will also include any resources shared with you.
* **Name** - The name for the new resource.

After filling out that information you can create the new resource. This will add a new resource to your resources list that you can then start/stop as needed.

## Allowing other users to clone your resource (Enterprise only)

For users of Saturn Cloud Enterprise, you can share your resource with other people in your organization. If you have a resource that you would like other users to be able to clone you can grant them permission to do so. Open the page of the resource and scroll to the "Sharing" card:

![Sharing card](/images/docs/share-card.png "doc-image")

Before another user can clone the resource, they first need to have the permission to do so. For the user you want to share with, select them from the drop down and then click **Add**. After they are added, they'll see the resource in the list of available resources when cloning. For convenience, you can send them a link to then clone the resource by sending them the URL on the card (the link only works after they have been added to the resource).