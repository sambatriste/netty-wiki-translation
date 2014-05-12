<div class="alert alert-danger"><strong>Looking for a tutorial?</strong> Visit [[the documentation home|Home]]. <strong>Got a question?</strong> Ask at <a href="https://stackoverflow.com/questions/tagged/netty">StackOverflow.com</a>.<br>Please note that this guide is <strong>not</strong> a 'user guide'.  It is not intended for the 'users' who build an application using Netty but for the contributors ('dev') who want to develop Netty itself.</div>

### Before getting started

* [[Set up your development environment.|Setting-up-development-environment]]
* [Read and sign the contributor license agreement](https://docs.google.com/spreadsheet/viewform?formkey=dHBjc1YzdWhsZERUQnhlSklsbG1KT1E6MQ) unless your contribution is trivial such as a single line change or a typo fix.

### Checklist

Please use the following checklist before pushing your commits or issuing a pull request:

* Does your work builds without any failure when you run '`mvn test`' from shell?
* Does your work introduce any new inspector warnings?
* Does your commit message or pull request description meet [[our commit message rules|Writing-a-commit-message]]?
* Did you [rebase](http://git-scm.com/book/en/Git-Branching-Rebasing) your pull request against the HEAD of the target branch and fix all conflicts?
* Did you sign [the contributor license agreement](https://docs.google.com/spreadsheet/viewform?formkey=dHBjc1YzdWhsZERUQnhlSklsbG1KT1E6MQ)?

### For contributors with push rights

* A pull request often contains multiple commits.  Those commits must be squashed into a small number of commits with explanatory comments.
* Do not merge a pull request via the web UI unless there's a good reason doing that. Refer to the 'Patch and Apply' section in [the Github help page](https://help.github.com/articles/using-pull-requests#merging-a-pull-request).
* Be careful when you [[update the official web site|Working-with-official-web-site]] or [[release a new version|Releasing-new-version]].