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

#### Upload all CSV files to S3
This will upload all .csv files in the current directory to S3
```
aws s3 sync . s3://s3-bucket-name/folder-name/ --exclude "*" --include "*.csv"
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

#### Shut down dev endpoints
shut down all dev endpoints in the account
```
import json
import boto3
import logging
import os

logger = logging.getLogger()
logger.setLevel(logging.INFO)

region_name = os.environ['AWS_REGION']
glue = boto3.client('glue',region_name=region_name)

def lambda_handler(event, context):
    
    
    devEndpointsList = glue.get_dev_endpoints(MaxResults=100)
    devEndpointsList = devEndpointsList['DevEndpoints']
    
    if len(devEndpointsList) > 0:
        endpointCount=0
        
        #loop through endpoints and shut them down
        for endpoint in devEndpointsList:
            endpointName = endpoint['EndpointName']
            deleteResponse = glue.delete_dev_endpoint(EndpointName=endpointName)
            print("Deleted endpoint '{}'.".format(endpointName))
            endpointCount+=1
        
        logger.info("{} Glue Dev Endpoint(s) have been deleted: '{}'".format(endpointCount,devEndpointsList))
    
        return {
            'statusCode': 200,
            'body': json.dumps("{} Glue Dev Endpoint(s) have been deleted: '{}'".format(endpointCount,devEndpointsList))
        }
        
    else:
        logger.info("No Glue Dev Endpoints are open at this time")
        return {
            'statusCode': 200,
            'body': json.dumps("No Glue Dev Endpoints are open at this time")
        }

```

#### Logging Boilerplate code
Use this in a lambda to easily log to CloudWatch
```
import logging
...
def log(name='aws_entity', logging_level=logging.INFO) -> logging.Logger:
    """Instantiate a logger
    """
    logger: logging.Logger = logging.getLogger(name)
    if len(logger.handlers) < 1:
        log_handler: logging.StreamHandler = logging.StreamHandler()
        formatter: logging.Formatter = logging.Formatter('%(levelname)-8s %(asctime)s %(name)-12s %(message)s')
        log_handler.setFormatter(formatter)
        logger.propagate = False
        logger.addHandler(log_handler)
        logger.setLevel(logging_level)
    return logger
```

### Python Lambda Boilderplate code
```python
import json
import boto3
import os
import logging
from botocore.exceptions import ClientError

# set logging
logger = logging.getLogger()
logger.setLevel(logging.INFO)

# boto3 resources
some_client = boto3.client('some_service')

# Envionrment Variables
ENV_VARIABLE = os.environ['ENV_VARIABLE']

def some_function(param1, param2):
    """
    Summary line. 
  
    Extended description of function. 
  
    Parameters: 
    arg1 (int): Description of arg1 
  
    Returns: 
    int: Description of return value 
    """
```

### Spark Glue Job Imports
```python
import sys
from awsglue.transforms import *
from awsglue.utils import getResolvedOptions
from pyspark.context import SparkContext
from awsglue.context import GlueContext
from awsglue.job import Job

glueContext = GlueContext(SparkContext.getOrCreate())
```
optional imports
```python
from pyspark.sql import SQLContext
from pyspark.sql.functions import *
from pyspark.sql.window import Window

sqlContext = SQLContext(SparkContext.getOrCreate())
```

Register data frame in sql context

```python
sqlContext.registerDataFrameAsTable(table_name.toDF(),'table_name')
new_table = spark.sql("select * from table_name")
```
