# Streamlit (Python)

To deploy a dashboard powered by [Streamlit](https://streamlit.io/), you'll need to do the following.

1. Create a [deployment](<docs/using-saturn-cloud/resources/deployments.md>).
2. Connect your deployment to a [git repo](<docs/using-saturn-cloud/gitrepo.md>) containing your Streamlit app.
3. Edit the deployment options in the following way:
  1. For the **Command** in the deployment options, select `streamlit run [filename]` where filename is the file with your streamlit app.
  2. In the **Environment Variables** options, add the environment variables below, which let Streamlit know how to use the correct ports with Saturn Cloud and be open to requests from other machines.

```
STREAMLIT_SERVER_PORT=8000
STREAMLIT_HEADLESS=True
STREAMLIT_SERVER_ADDRESS=0.0.0.0
```

With that, when you start the deployment and go to the URL you should see your dashboard.
