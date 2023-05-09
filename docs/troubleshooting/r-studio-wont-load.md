# R Studio won't load

Sometimes R studio won't load. This article refers to situations where your R server starts up without issue, however when you load R studio in the browser you cannot load it. This is usually due to R studio attempting to re-load a very large workspace.

The immediate fix is to move/delete your files such that R studio will not attempt to re-load the workspace. This can be accomplished by dropping the following snippet into your resource's start script.

```
export TS=`date +%s`
export BACKUP="/home/jovyan/BACKUP-${TS}"
echo "BACKING UP RSTUDIO FILES TO ${BACKUP}"
mkdir -p ${BACKUP}
mv ~/.rstudio-data ${BACKUP}
mv ~/.rstudio-database ${BACKUP}
mv ~/.local ${BACKUP}
```

Long term, you may want to [disable automatic workspace loading in R studio.](https://community.rstudio.com/t/defaults-of-saving-and-restoring-workspace/939)
