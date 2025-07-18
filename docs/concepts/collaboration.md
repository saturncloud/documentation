# Collaboration and Teams

### Providing resource templates for teams

One of the most common use cases for Saturn Cloud is to make it easy for data scientists to create resources with the "right" settings for the company or team that they work for. This usually means ensuring that they load code from the appropriate Git repositories, start with images that have all the necessary dependencies installed, and have data access credentials pre-loaded.

Saturn cloud recipes encapsulate all properties of a Saturn Cloud resource as a JSON file. A Resource template just stores a copy of this JSON, and uses it to bootstrap team resources to have the right settings by default. This speeds up new employee onboarding.

### Sharing assets

In addition to sharing resource templates, the components of a resource can be shared with the team as well. These include

- Docker Images - preloaded configuration with the software your team uses
- Secrets - data access credentials that are encrypted and can be attached to resources
- Shared folders - NFS volume mounts

Each of these assets can be owned by a user, a group (a collection of users) or the entire organization. That ownership determines who the resource is shared with.

### Real time collaboration

At Saturn Cloud we do not focus on real time collaboration. For most users, collaborating with Git is more productive. However Jupyter does have experimental support for real time collaboration. Please talk to us if you'd like to enable this.

### Managing production

Resources can be owned by users as well as groups. This enables a team to create a set of production resources, that can all be managed by members of the team. A common pattern is to start with the resource being owned by an individual data scientist, and then promote it to be owned by the group when it becomes important (when you want team-mates to be able to manage it while someone is on vacation)
