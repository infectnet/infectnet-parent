# InfectNet Parent

Parent project that does not contain any code, but description of the common development policies among other repositories.

Child repositories:
 * [InfectNet Server](https://github.com/infectnet/infectnet-server)
 * [InfectNet Browser Frontend](https://github.com/infectnet/infectnet-browser-frontend)

You can access the InfectNet Waffle board at [here](https://waffle.io/infectnet/infectnet-parent/).

[![Throughput Graph](https://graphs.waffle.io/infectnet/infectnet-parent/throughput.svg)](https://waffle.io/infectnet/infectnet-parent/metrics/throughput)

## Workflow

InfectNet uses [waffle.io](waffle.io) and a downscaled version of [gitflow](https://github.com/nvie/gitflow).

### gitflow

Since InfectNet is only developed by a team of three, there's no need for subteam fetches (see [Decentralized but centralized](http://nvie.com/posts/a-successful-git-branching-model/#decentralized-but-centralized)). Therefore we're ignoring that part of `gitflow`.

Apart from that, other features are used as follows:

 * The `master` branch only contains release commits, except the first initialization related commits.
 * There are no direct commits to the `develop` branch, only merges from feature branches and release branches.
 * A new feature branch is created for every `waffle` task.

### waffle

The [recommended waffle workflow](https://github.com/waffleio/waffle.io/wiki/Recommended-Workflow-Using-Pull-Requests-&-Automatic-Work-Tracking) is used.

### All together now

Steps of developing a new feature:

1. Add an issue to the `waffle` board. This creates a GitHub issue and a card in the **Backlog** column.
    * Assign descriptive labels to the task so it is easier to decide whether it is important or not.
1. When the feature is ready to be worked on, move the card to the **Ready** column.
    * If appropriate, assign a developer to the task.
1. Create a new feature branch branched off from `develop`. The name of the newly created branch must be similar to the name of the task. It must be hyphenated and must contain the number of the corresponding issue too. Example: `new-feature-#21`
1. When finished create a new Pull Request. The name of the pull request must contain the number of the corresponding issue in the following format: `<feature name> - closes #<issue number>`. Example: `New feature - closes #21` After the PR is created there are some more steps to do:
    1. Fix all errors (Travis or Codacy).
    1. Wait for at least one other developer to review your work.
    1. Fix merge conflicts.
1. If everything's green and other developers accept your work then you can merge the Pull Request and delete the branch. Also the `waffle` card should be moved to the **Done** column.

## Files

### TODO.md
Things that should be done and exited the `idea` phase.

### IDEA.md
Good ideas that should not be forgotten and should be taken into account when developing.

### TECHS.md
The different technologies to be used throughout the project.
