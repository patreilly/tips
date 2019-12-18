## Tips
This is a collection of useful scripts and code snippets I use that help me with regular tasks.


#### Generate cloudformation parameters file.
After spending hours on a cloudformation template, the last thing I want to do is create the parameters file. So I used this.
```
aws cloudformation get-template-summary --template-body file://cloudformation-template.yml --profile pat | jq '[.Parameters | .[] | {"ParameterKey": .ParameterKey, "ParameterValue": .DefaultValue}]' > params.json
```

#### Push to multiple git repos
I'm often working in multiple git repos for customers and need to make sure two of them are in sync at the same time. I configure git like the following to accomplish that.
```
[remote "all"]
        url = https://code.corp.coderepo.com/mrdevperson/code-project.git
        url = https://gitlab.com/codereponumbertwo/code-project-copy.git
```
to push to both repos at the same time, I use:
```
git push all
```