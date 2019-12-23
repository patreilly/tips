## Tips
This is a collection of useful scripts and code snippets I use that help me with regular tasks.

### CloudFormation

#### Generate cloudformation parameters file.
After spending hours on a cloudformation template, the last thing I want to do is create the parameters file. So I used this.
```
aws cloudformation get-template-summary --template-body file://cloudformation-template.yml --profile pat | jq '[.Parameters | .[] | {"ParameterKey": .ParameterKey, "ParameterValue": .DefaultValue}]' > params.json
```
#### Get latest AMI in Region
Windows:
```
aws ssm get-parameters --names /aws/service/ami-windows-latest/Windows_Server-2016-English-Full-Base --region us-west-2 
```

Linux:
```
aws ssm get-parameters --names /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2 --region us-west-2 
```
### Productivity

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

### Data Engineering

#### Spark Glue Job, First Run Problems
When your pyspark script is doing delta inserts by first checking the existing records in the S3 bucket, the first time the job runs is going to fail unless you account for it correctly. This solution checks the existence of the table in the glue catalog first:

```
# Check if this is the first job run
# by checking for this catalog table in Glue Catalog
tables = glue_client.get_tables(
    DatabaseName='glue_database_name'
    )

table_list=[]
for t in tables['TableList']:
    table_list+=t['Name']

is_first_job_run=True
if 'table_name_to_check' in table_list:
    is_first_job_run=False


# if this is the first job run, just insert records without checking
if is_first_job_run==True:
    # insert records without checking existing ones
else:
    # do normal delta insert
```