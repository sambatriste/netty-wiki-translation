<div class="alert alert-danger"><strong>Looking for a tutorial?</strong> Visit [[the documentation home|Home]]. <strong>Got a question?</strong> Ask at <a href="https://stackoverflow.com/questions/tagged/netty">StackOverflow.com</a>.<br>Please note that this guide is <strong>not</strong> a 'user guide'.  It is not intended for the 'users' who build an application using Netty but for the contributors ('dev') who want to develop Netty itself.</div>

## Set up your development environment

See [[here|Setting-up-development-environment]].

## Write a meaningful commit message

See [[here|Writing-a-commit-message]].

## Sign the contributor license agreement

Please read and sign [the contributor license agreement](https://docs.google.com/spreadsheet/viewform?formkey=dHBjc1YzdWhsZERUQnhlSklsbG1KT1E6MQ) unless your contribution is trivial such as a single line change or a typo fix.

## Checklist

Please use the following checklist before pushing your commits or issuing a pull request:

* Does your work builds without any failure when you run '`mvn test`' from shell?
* Does your work introduce any new inspector warnings?
* Do your commit messages follow the format mentioned in the 'writing a meaningful commit message' section above?
* Did you sign the contributor license agreement?

## Work with a pull request

### Issue a pull request

Pull requests should be targeted at the branch for the latest stable releases.  If the pull request is for fixing a bug which also affects an old branch like `3.x`, we recommend you to submit another pull request for that branch, too.

1. [Rebase](http://git-scm.com/book/en/Git-Branching-Rebasing) your changes against the upstream branch.  Resolve any conflicts that arise.
1. Write JUnit test cases if possible. If not sure about how to write one, ask us how to write one.
1. Run `mvn test` before the initial submission or the subsequent pushes, and ensure the build succeeds.

### Merge a pull request

A pull request often contains multiple commits.  Those commits must be squashed into a small number of commits with explanatory comments.

Do not merge a pull request via the web UI unless there's a good reason doing that. Refer to the 'Patch and Apply' section in [the Github help page](https://help.github.com/articles/using-pull-requests#merging-a-pull-request).

## Update the official web site

See [[here|Working-with-official-web-site]].

## Release a new version

See [[here|Releasing-new-version]].

## Design ideas

* [[Thread model in 4.x|Thread model]]