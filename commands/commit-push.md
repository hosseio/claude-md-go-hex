# description: Create conventional commit and push, optionally create PR with --pr

/commit --push $ARGUMENTS

If `--pr` flag is present in arguments, after the commit and push completes successfully, execute `/pr` to create a pull request.
