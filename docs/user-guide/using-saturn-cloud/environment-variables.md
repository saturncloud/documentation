# Environment Variables

Environment variables are values that you can set within a machine that all running processes have access to. Compared to values set within R or Python which can only be accessed by that running process, environment variables
are more universally accessible. They are also a useful way to set values for your code that can be changed without having to change the code itself--you wouldn't need to make a git commit to update the value.

Environment variables can be set in two different ways in Saturn Cloud, depending on if the environment variable has secret information you want to hide or not.

### Set a non-secure environment variable for a single resource

The **Environment Settings** section of the resource settings has an area where you can add environment variables. These variables will be accessible by that resource only, and anyone who has access to the resource
will be able to see the value of the variable, including if you clone it. You can add the environment variables as `VARIABLE=value`, where `VARIABLE` is the name of the variable and `value` is the value of the variable.

![Environment variables UI](/images/docs/environment.webp "doc-image")

### Set a secret shared by multiple resources

You may have environment variables for things like passwords that you want to ensure stay secure, and often you want to share across multiple resources. To set these, use the [**Secrets**](<docs/user-guide/using-saturn-cloud/secrets.md>) tab of Saturn Cloud.

## Using environment values

{{% alert %}}
Note: If the resource is already running when you created an environment variable or secret, you will need to restart the resource before you can use them.
{{% /alert %}}

How to access an environment variable depends on the programming language you are using. Here are examples of how to the environment variables in commonly used languages in Saturn Cloud,
but these directions are not Saturn Cloud specific. For other languages you can check the documentation for that particular language. These examples are for a variable named `VARIABLE`:

* **Python** - `os.environ["VARIABLE"]`
* **R**  - `Sys.getenv("VARIABLE")`
* **Julia**  - `ENV["VARIABLE"]`