# API and Recipes

## The API

Everything that can be done in the UI can be done via Saturn Cloud API calls. The [API](https:///api.saturncloud.io) is what our UI uses to orchestrate resources on the backend - as a result it is pretty low level. For most purposes, Saturn Cloud recipes are more appropriate

## Recipes

Saturn Cloud recipes are JSON representation of objects in Saturn Cloud. Every resource in Saturn Cloud can be exported as a recipe and Saturn Cloud recipes can be used to import resources as well as share them with colleagues.

### Infrastructure as code

Saturn Cloud recipes can also be used for infrastructure as code patterns. Recipes, which are a declarative representation of Saturn Clodu resources can be applied to Saturn Cloud, which will create a resource if it does not exist. If the resource has already been created, this will update it to the new settings.

## Integration with CI/CD

Saturn Cloud resources are transparent. As a result, it is easy to orchestrate Saturn Cloud from standard CI/CD tools like GitHub actions. There are a few common patterns

### Building Images

One common use case is to store image definitions (Dockerfile and environment.yaml) in Git, and then have CI/CD pipelines build those images and register them with Saturn Cloud for use by your data science team.

### Git Ops

Another common use case is to store recipe definitions in Git, and have CI/CD pipelines apply them to Saturn Cloud resources. This enables you to manage production application state from your CI/CD pipeline.
