After making changes on Netty, please make sure your commit message contains enough contextual information so that anyone understands the intention of the changes.  Unless the commit is trivial, please use the following form (by the courtesy of Steve Gury et al):

```plain
One line description of your change
 
Motivation:

Explain here the context, and why you're making that change.
What is the problem you're trying to solve.
 
Modifications:

Describe the modifications you've done.
 
Result:

After your change, what will change.
```
You can add the above message as a git commit template if you like. To do so, create a file with the contents above but add a ```#``` to each line. Doing this stops git from committing as all lines will be comments. You can then issue the following git command:
```
git config commit.template ~/.git.commit.template
```
Where ```~/.git.commit.template``` can be any file path you wish.
