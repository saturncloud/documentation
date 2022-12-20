# Embed Workspaces Outside of Saturn Cloud


In certain situations you may want to grant access to a Saturn Cloud Jupyter Server or R Server without having the full Saturn Cloud user interface. Instead, you may want to be able to create a resource, start and stop it, and open the IDE from some other website. This guide is to set up such an environment. It falls into two steps: creating a resource programmatically for a users, then settings up the start, stop, open IDE buttons so the user can use the resource.

Before following the steps below you'll need the following:

* The **base URL** of your Saturn Cloud application. It should be in the format `https://app.{yourorg}.saturnenterprise.io`
without a trailing slash.
* An **Saturn Cloud admin user token**. This can be found by logging into Saturn Cloud as an admin and going to the settings page. This user token will be used to perform most of the actions.

_Finally, note that the Saturn Cloud API is constantly improving, and some of these steps may become obsolete as we streamline this process. If any step doesn't work, please reach out to us at [support@saturncloud.io](mailto:support@saturncloud.io)._

## Creating a resource programmatically

These steps will show how to create a resource for an arbitrary user. Note that the user must already exist for these steps to work. Creating the user themselves is outside the scope of this document.

To create a resource for a user, we need to first need to get a user token on behalf of the user, then we need to create a resource from a recipe using that token.

In addition to the base URL and admin token, before starting this process, you'll also need:

* A **username** to create a resource for
* A **recipe** json file to use. These files fully describe Saturn Cloud resources. The full schema can be found on the [Saturn Cloud GitHub](https://github.com/saturncloud/recipes).

### Getting a token on behalf of the user

First, create a login link for the user in question using the following curl command:

```shell
curl --location --request POST '{base_url}/auth/createloginlink/{username}' \
--header 'Authorization: token {admin_token}'
```

The response to this request will include a temporary login link. _In a session created with that link_, use the following command to then get the user token:

```shell
curl --location --request GET '{base_url}/api/user/token'
```

### Creating the resource

Once you have the user token, you can then make a resource for that user. Use a POST request to send the recipe for the resource to Saturn Cloud:

```shell
curl --location --request POST '{base_url}/api/resource_recipes/clone' \
--header 'Authorization: token {user_token}' \
--header 'Content-Type: application/json' \
--data-raw '{recipe}'
```

The response will include an `id` field. We'll refer to that as the `resource_id` since it can be used to uniquely refer to the resource.
This should be stored and is necessary for the second half of this walkthrough.

## Using a workspace resource programmatically

Once a user has a workspace resource, as in a Jupyter Server or R Server resource with an IDE, you can then create controls for it outside of the Saturn Cloud UI. To use a Saturn Cloud workspace resource without a UI, there are for actions you'll want to allow users to do:

1. Start a resource
2. Stop a resource
3. Get the state of a resource to know if it's online
4. Open the IDE of a running resource

In addition to the base url and admin token, to do these actions you'll need the _resource_id_ (which was returned when creating the resource).

### Starting a resource

To start a resource, use the following POST command:

```shell
curl --location --request POST '{base_url}/api/workspaces/{resource_id}/start' \
--header 'Authorization: token {admin_token}'
```

### Stopping a resource

To stop a resource, use the following POST command:

```shell
curl --location --request POST '{base_url}/api/workspaces/{resource_id}/stop' \
--header 'Authorization: token {admin_token}'
```

### Getting the state of a resource

To get the state of a resource, use the following GET command:

```shell
curl --location --request GET '{base_url}/api/workspaces/{resource_id}' \
--header 'Authorization: token {admin_token}'
```

The response will include a field `status` which gives the status of the resource. It should be one of:

* `running` - The resource is online and available to use.
* `stopped` - The resource is offline.
* `pending` - The resource is starting to come online but is not ready.
* `error` - An issue occurred when starting the resource.

This endpoint should not be hit more that once every ten seconds. We recommend polling at that rate when the status is pending to quickly determine when the resource is ready. Otherwise, the status can be polled at a lower rate as it is unlikely to change without user intervention.

### Opening the IDE of the running resource

The same request for the status of a resource will also provide the URL to the IDE, provided the resource is running:

```shell
curl --location --request GET '{base_url}/api/workspaces/{resource_id}' \
--header 'Authorization: token {admin_token}'
```

The `url` field in the response a link to the IDE. The user who owns the resource will be able to click this to enter the IDE. Note however, that the user must be logged in to access the IDE--this link will only work for the user who owns the resource.
