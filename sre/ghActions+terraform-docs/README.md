# GH Actions + terraform-docs.

***Terraform Docs*** is a tool to generate terraform documentation automaticaly.

[Learn more about terraform-docs here.](https://terraform-docs.io/user-guide/introduction/)

The goal in this project is to use ***GitHub Actions*** to automatically run terraform-docs on every push or PR to any specified branch, in that way you can standarize terraform `readme.md` files and generate or verified that are updated.

## Resoruces

There is already a GH action implemented along terraform-docs just to accomplish this purpose.

[Here you can find it.](https://github.com/terraform-docs/gh-actions)

That project has a list of available properties that are quite useful to personalize the generation of the documentation.

## Problematic

However, while working with several directories containing different terraform modules and in order to applied to all of them the easiest way to do it would be to set the `find-dir` property, which will get a list of directories with terraform code inside.

The problem what that due the architecture of the project some of that directories had nested directories on them which didn't need terraform-docs to apply. 

## Solution

The solution was to use `working-dir` intead of `find-dir` and do the command to manually extract just the wanted directories. In order to do that you need to run the command on the first step and save the output as an environment variable to be able to use it on a different step later on. 

[Here you can learn how to do that.](https://docs.github.com/en/actions/reference/workflow-commands-for-github-actions#environment-files)


At the end it should look like this:

    name: Generate terraform documentation
    on:
    pull_request:
        branches: 
        - main
    jobs:
    docs:
        runs-on: ubuntu-latest
        steps:
        - uses: actions/checkout@v2
            with:
            ref: ${{ github.event.pull_request.head.ref }}
        - name: make env bar
            run: |
            string="dirs_list=";for dir in $(ls -p | grep / | sed 's|/$||'); do ls "$dir/"*".tf" &>/dev/null && string+="$dir",; done; echo $string | sed 's|,$||' >>$GITHUB_ENV
        - name: Render terraform docs and push changes back to PR
            uses: terraform-docs/gh-actions@v0.6.1
            with:
            working-dir: ${{ env.dirs_list }}
            config-file: ../.terraform-docs.yml
            output-file: README.md
            git-push: "true"
            git-commit-message: "Automated README.md generation"% 

*Note:* You can noticed that it is using just one `terraform-docs-yml` for all the modules.

