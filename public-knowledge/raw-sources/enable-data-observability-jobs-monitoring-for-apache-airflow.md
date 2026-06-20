---
id: B636CA9F-F5DB-4123-9648-4BB8A4E272A8
title: Enable Data Observability: Jobs Monitoring for Apache Airflow
status: draft
created-at: 2026-06-20T00:23:27Z
updated-at: 2026-06-20T00:23:27Z
---

# Enable Data Observability: Jobs Monitoring for Apache Airflow

Title: Enable Data Observability: Jobs Monitoring for Apache Airflow
Site: Datadog Infrastructure and Application Monitoring
Byline: Datadog
URL: https://docs.datadoghq.com/data_observability/jobs_monitoring/airflow/
Content-Type: text/html; charset=utf-8
Extraction: readability

This product is not supported for your selected Datadog site. ().

Data Observability: Jobs Monitoring provides visibility into the performance and reliability of workflows run by Apache Airflow DAGs.

Requirements

Apache Airflow 2.7 or later, including Airflow 3

apache-airflow-providers-openlineage

Setup

To get started, follow the instructions below.

Install openlineage provider for both Airflow schedulers and Airflow workers by adding the following into your requirements.txt file or wherever your Airflow dependencies are managed:

apache-airflow-providers-openlineage

Configure openlineage provider. Choose one of the following configuration options and set the environment variables, making them available to pods where you run Airflow schedulers and Airflow workers:

Option 1: Datadog Transport (Recommended)

Requirements: Requires apache-airflow-providers-openlineage version 2.7.3 or later and openlineage-python version 1.37.0 or later.

export DD_API_KEY=<DD_API_KEY>
export DD_SITE=<DD_SITE>
export OPENLINEAGE__TRANSPORT__TYPE=datadog
# OPENLINEAGE_NAMESPACE sets the 'env' tag value in Datadog. You can hardcode this to a different value
export OPENLINEAGE_NAMESPACE=${AIRFLOW_ENV_NAME}

Replace <DD_API_KEY> with your valid Datadog API key.

Replace <DD_SITE> with your Datadog site (for example, ).

Option 2: Composite Transport

Requirements: Requires apache-airflow-providers-openlineage version 1.11.0 or later and openlineage-python version 1.37.0 or later.

Use this option if you’re already using OpenLineage with another system and want to add Datadog as an additional destination. The composite transport sends events to all configured transports.

For example, if you’re using an HTTP transport to send events to another system:

# Your existing HTTP transport configuration
export OPENLINEAGE__TRANSPORT__TYPE=composite
export OPENLINEAGE__TRANSPORT__TRANSPORTS__EXISTING__TYPE=http
export OPENLINEAGE__TRANSPORT__TRANSPORTS__EXISTING__URL=<YOUR_EXISTING_URL>
export OPENLINEAGE__TRANSPORT__TRANSPORTS__EXISTING__AUTH__TYPE=api_key
export OPENLINEAGE__TRANSPORT__TRANSPORTS__EXISTING__AUTH__API_KEY=<YOUR_EXISTING_API_KEY>

# Add Datadog as an additional transport
export DD_API_KEY=<DD_API_KEY>
export DD_SITE=<DD_SITE>
export OPENLINEAGE__TRANSPORT__TRANSPORTS__DATADOG__TYPE=datadog
# OPENLINEAGE_NAMESPACE sets the 'env' tag value in Datadog. You can hardcode this to a different value
export OPENLINEAGE_NAMESPACE=${AIRFLOW_ENV_NAME}

Replace <DD_API_KEY> with your valid Datadog API key.

Replace <DD_SITE> with your Datadog site (for example, ).

Replace <YOUR_EXISTING_URL> and <YOUR_EXISTING_API_KEY> with your existing OpenLineage transport configuration.

In this example, OpenLineage events are sent to both your existing system and Datadog. You can configure multiple transports by giving each one a unique name (like EXISTING and DATADOG in the example above).

Option 3: Simple Configuration

This option uses the URL-based configuration and works with all versions of the OpenLineage provider:

export OPENLINEAGE_URL=<DD_DATA_OBSERVABILITY_INTAKE>
export OPENLINEAGE_API_KEY=<DD_API_KEY>
# OPENLINEAGE_NAMESPACE sets the 'env' tag value in Datadog. You can hardcode this to a different value
export OPENLINEAGE_NAMESPACE=${AIRFLOW_ENV_NAME}

Replace <DD_DATA_OBSERVABILITY_INTAKE> with https://data-obs-intake..

Replace <DD_API_KEY> with your valid Datadog API key.

If you’re using Airflow v2.7 or v2.8, also add these two environment variables along with the previous ones. This fixes an OpenLineage config issue fixed at apache-airflow-providers-openlineage v1.7, while Airflow v2.7 and v2.8 use previous versions.

#!/bin/sh
# Required for Airflow v2.7 & v2.8 only
export AIRFLOW__OPENLINEAGE__CONFIG_PATH=""
export AIRFLOW__OPENLINEAGE__DISABLED_FOR_OPERATORS=""

Check official documentation configuration-openlineage for other supported configurations of the openlineage provider.

Trigger an update to your Airflow pods and wait for them to finish.

Optionally, set up log collection for correlating task logs to DAG run executions in Data Observability: Jobs Monitoring. Correlation requires the logs directory to follow the default log filename format.

Note: For log correlation to work, inject Airflow task context attributes (DAG ID, run ID, task ID, attempt number) into task logs as structured facets in Datadog. See Inject Airflow Task Context into Logs.

The PATH_TO_AIRFLOW_LOGS value is $AIRFLOW_HOME/logs in standard deployments, but may differ if customized. Add the following annotation to your pod:

ad.datadoghq.com/base.logs: '[{"type": "file", "path": "PATH_TO_AIRFLOW_LOGS/*/*/*/*.log", "source": "airflow"}]'

Adding "source": "airflow" enables the extraction of the correlation-required attributes by the Airflow integration logs pipeline.

These file paths are relative to the Agent container. Mount the directory containing the log file into both the application and Agent containers so the Agent can access it. For details, see Collect logs from a container local log file.

Note: Log collection requires the Datadog Agent to already be installed on your Kubernetes cluster. If you haven’t installed it yet, see the Kubernetes installation documentation.

For more methods to set up log collection on Kubernetes, see the Kubernetes and Integrations configuration section.

Validation

In Datadog, view the Data Observability: Jobs Monitoring page to see a list of your Airflow job runs after the setup.

Troubleshooting

Set OPENLINEAGE_CLIENT_LOGGING to DEBUG along with the other environment variables set previously for OpenLineage client and its child modules. This can be useful in troubleshooting during the configuration of openlineage provider.

To run an automated check of your OpenLineage setup, see Troubleshoot Airflow Setup with the OpenLineage Validation DAG.

Requirements

Apache Airflow 2.7.0 or later, including Airflow 3

apache-airflow-providers-openlineage

Setup

If you are using Airflow 2.7.2, 2.8.1, or 2.9.2: MWAA default constraints pin older apache-airflow-providers-openlineage` versions. These versions include known issues that can degrade the Data Observability experience. To upgrade to provider versions with fixes, see Upgrade OpenLineage provider on Amazon MWAA for Airflow 2.7.2, 2.8.1, and 2.9.2.

To get started, follow the instructions below.

Install openlineage provider by adding the following into your requirements.txt file:

apache-airflow-providers-openlineage

Configure openlineage provider. The simplest option is to set the following environment variables in your Amazon MWAA start script:

#!/bin/sh
export OPENLINEAGE_URL=<DD_DATA_OBSERVABILITY_INTAKE>
export OPENLINEAGE_API_KEY=<DD_API_KEY>
# AIRFLOW__OPENLINEAGE__NAMESPACE sets the 'env' tag value in Datadog. You can hardcode this to a different value
export AIRFLOW__OPENLINEAGE__NAMESPACE=${AIRFLOW_ENV_NAME}

Replace <DD_DATA_OBSERVABILITY_INTAKE> fully with https://data-obs-intake..

Replace <DD_API_KEY> fully with your valid Datadog API key.

If you’re using Airflow v2.7 or v2.8, also add these two environment variables to the startup script. This fixes an OpenLineage config issue fixed at apache-airflow-providers-openlineage v1.7, while Airflow v2.7 and v2.8 use previous versions.

#!/bin/sh
# Required for Airflow v2.7 & v2.8 only
export AIRFLOW__OPENLINEAGE__CONFIG_PATH=""
export AIRFLOW__OPENLINEAGE__DISABLED_FOR_OPERATORS=""

Check official documentation configuration-openlineage for other supported configurations of openlineage provider.

Deploy your updated requirements.txt and Amazon MWAA startup script to your Amazon S3 folder configured for your Amazon MWAA Environment.

Optionally, set up Log Collection for correlating task logs to DAG run executions in DJM:

Configure Amazon MWAA to send logs to CloudWatch.

Send the logs to Datadog.

Validation

In Datadog, view the Data Observability: Jobs Monitoring page to see a list of your Airflow job runs after the setup.

Troubleshooting

Ensure your Execution role configured for your Amazon MWAA Environment has the right permissions to the requirements.txt and Amazon MWAA start script. This is required if you are managing your own Execution role and it’s the first time you are adding those supporting files. See official guide Amazon MWAA execution role for details if needed.

Set OPENLINEAGE_CLIENT_LOGGING to DEBUG in the Amazon MWAA start script for OpenLineage client and its child modules. This can be useful in troubleshooting during the configuration of openlineage provider.

To run an automated check of your OpenLineage setup, see Troubleshoot Airflow Setup with the OpenLineage Validation DAG.

Requirements

Astro Runtime 12.1.0+

apache-airflow-providers-openlineage 1.11.0+

openlineage-python 1.23.0+

Setup

To set up the OpenLineage provider, define the following environment variables. You can configure these variables in your Astronomer deployment using either of the following methods:

From the Astro UI: Navigate to your deployment settings and add the environment variables directly.

In the Dockerfile: Define the environment variables in your Dockerfile to ensure they are included during the build process.

OPENLINEAGE__TRANSPORT__TYPE=composite
OPENLINEAGE__TRANSPORT__TRANSPORTS__DATADOG__TYPE=http
OPENLINEAGE__TRANSPORT__TRANSPORTS__DATADOG__URL=<DD_DATA_OBSERVABILITY_INTAKE>
OPENLINEAGE__TRANSPORT__TRANSPORTS__DATADOG__AUTH__TYPE=api_key
OPENLINEAGE__TRANSPORT__TRANSPORTS__DATADOG__AUTH__API_KEY=<DD_API_KEY>
OPENLINEAGE__TRANSPORT__TRANSPORTS__DATADOG__COMPRESSION=gzip

replace <DD_DATA_OBSERVABILITY_INTAKE> with https://data-obs-intake..

replace <DD_API_KEY> with your valid Datadog API key.

Optional:

Set AIRFLOW__OPENLINEAGE__NAMESPACE with a unique name for the env tag on all DAGs in the Airflow deployment. This allows Datadog to logically separate this deployment’s jobs from those of other Airflow deployments.

Set OPENLINEAGE_CLIENT_LOGGING to DEBUG for the OpenLineage client and its child modules to log at a DEBUG logging level. This can be useful for troubleshooting during the configuration of an OpenLineage provider.

See the Astronomer official guide for managing environment variables for a deployment. See Apache Airflow’s OpenLineage Configuration Reference for other supported configurations of the OpenLineage provider.

Trigger an update to your deployment and wait for it to finish.

Validation

In Datadog, view the Data Observability: Jobs Monitoring page to see a list of your Airflow job runs after the setup.

Troubleshooting

Check that the OpenLineage environment variables are correctly set on the Astronomer deployment.

Note: Using the .env file to add the environment variables does not work because the variables are only applied to the local Airflow environment.

To run an automated check of your OpenLineage setup, see Troubleshoot Airflow Setup with the OpenLineage Validation DAG.

Data Observability: Jobs Monitoring for Airflow is not yet compatible with Dataplex data lineage. Setting up OpenLineage for Data Observability: Jobs Monitoring overrides your existing Dataplex transport configuration.

Requirements

Cloud Composer 2 or later

apache-airflow-providers-openlineage

Setup

To get started, follow the instructions below.

In the Advanced Configuration tab, under Airflow configuration override, click Add Airflow configuration override and configure these settings:

In Section 1, enter openlineage.

In Key 1, enter disabled.

In Value 1, enter False to make sure OpenLineage is activated.

In Section 2, enter openlineage.

In Key 2, enter transport.

In Value 2, enter the following:

{
"type": "http",
"url": "<DD_DATA_OBSERVABILITY_INTAKE>",
"auth": {
"type": "api_key",
"api_key": "<DD_API_KEY>"
}
}

Replace <DD_DATA_OBSERVABILITY_INTAKE> fully with https://data-obs-intake..

Replace <DD_API_KEY> fully with your valid Datadog API key.

Optional: Configure the OpenLineage namespace to set the env tag value in Datadog:

In Section 3, enter openlineage.

In Key 3, enter namespace.

In Value 3, enter your Composer environment value (for example, prod, dev, staging, or test).

Note: If Dataplex is enabled, the namespace is already set by default to the Composer environment name. Manually setting the OpenLineage namespace here optionally allows you to override this default value.

Check official Airflow and Composer documentation pages for other supported configurations of the openlineage provider in Google Cloud Composer.

Validation

In Datadog, view the Data Observability: Jobs Monitoring page to see a list of your Airflow job runs after the setup.

Troubleshooting

Set OPENLINEAGE_CLIENT_LOGGING to DEBUG in the Environment variables tab of the Composer page for OpenLineage client and its child modules. This can be useful in troubleshooting as you configure the openlineage provider.

To run an automated check of your OpenLineage setup, see Troubleshoot Airflow Setup with the OpenLineage Validation DAG.

Advanced Configuration

Link your dbt jobs with Airflow tasks

The lineage_root_* macros used below require apache-airflow-providers-openlineage 2.3.0 or later. On older provider versions (for example, Airflow 2.9.2 with provider 2.2.0), see Backport OpenLineage lineage macros for older provider versions.

You can monitor your dbt jobs that are running in Airflow by connecting the dbt telemetry with respective Airflow tasks, using OpenLineage dbt integration.

To see the link between Airflow tasks and dbt jobs, follow those steps:

Install openlineage-dbt. Reference Using dbt with Amazon MWAA to setup dbt in the virtual environment.

pip3 install openlineage-dbt>=1.36.0

Change the dbt invocation to dbt-ol (OpenLineage wrapper for dbt).

Also, add the –consume-structured-logs flag to view dbt jobs while the command is still running.

dbt-ol run --consume-structured-logs --project-dir=$TEMP_DIR --profiles-dir=$PROFILES_DIR

In your DAG file, set the OpenLineage parent and root-parent environment variables on the Airflow task that runs the dbt process so dbt jobs link to both the immediate parent task and the outermost DAG run in Datadog:

dbt_run = BashOperator(
task_id="dbt_run",
dag=dag,
bash_command=f"dbt-ol run --consume-structured-logs --project-dir=$TEMP_DIR --profiles-dir=$PROFILES_DIR",
append_env=True,
env={
"OPENLINEAGE_PARENT_JOB_NAMESPACE": "{{ lineage_job_namespace() }}",
"OPENLINEAGE_PARENT_JOB_NAME": "{{ lineage_job_name(task_instance) }}",
"OPENLINEAGE_PARENT_RUN_ID": "{{ lineage_run_id(task_instance) }}",
"OPENLINEAGE_ROOT_PARENT_JOB_NAMESPACE": "{{ lineage_root_job_namespace(task_instance) }}",
"OPENLINEAGE_ROOT_PARENT_JOB_NAME": "{{ lineage_root_job_name(task_instance) }}",
"OPENLINEAGE_ROOT_PARENT_RUN_ID": "{{ lineage_root_run_id(task_instance) }}",
},
)

Link your Spark jobs with Airflow tasks

The lineage_root_* macros require apache-airflow-providers-openlineage 2.3.0 or later. On older provider versions (for example, Airflow 2.9.2 with provider 2.2.0), see Backport OpenLineage lineage macros for older provider versions.

OpenLineage integration can automatically inject Airflow’s parent job information (namespace, job name, run id) into Spark application properties. This creates a parent-child relationship between Airflow tasks and Spark jobs, enabling you to troubleshoot both systems in one place.

Note: This feature requires apache-airflow-providers-openlineage version 2.1.0 or later (supported from Airflow 2.9+).

Verify operator compatibility: Check the Apache Airflow OpenLineage documentation to confirm your Spark operators are supported. This feature only works with specific operators like SparkSubmitOperator and LivyOperator.

Make sure your Spark jobs are actively monitored through Data Observability: Jobs Monitoring.

Enable automatic parent job information injection by setting the following configuration:

AIRFLOW__OPENLINEAGE__SPARK_INJECT_PARENT_JOB_INFO=true

This automatically injects parent job properties for all supported Spark Operators. To disable for specific operators, set openlineage_inject_parent_job_info=False on the operator.

Manually inject parent job information for unsupported operators

For operators that do not support automatic injection (for example, a BashOperator running the AWS CLI to start an Amazon EMR Serverless job), use the OpenLineage Airflow lineage macros to pass parent and root-parent identifiers as Spark configuration properties:

from datetime import datetime

from airflow import DAG
from airflow.operators.bash import BashOperator

with DAG(
dag_id="emr_serverless_with_openlineage_parent",
start_date=datetime(2026, 1, 1),
schedule_interval=None,
catchup=False,
) as dag:

submit = BashOperator(
task_id="start_emr_serverless_job",
bash_command=r"""
set -euo pipefail

aws emr-serverless start-job-run \
--application-id <APPLICATION_ID> \
--execution-role-arn <EXECUTION_ROLE_ARN> \
--name <JOB_NAME> \
--region <AWS_REGION> \
--mode STREAMING \
--job-driver '{"sparkSubmit": {
"entryPoint": "<JAR_URL>",
"entryPointArguments": ["--config-key", "<CONFIG_KEY>", "--stage", "<STAGE>"],
"sparkSubmitParameters": "\
--conf spark.app.name=<APP_NAME> \
--conf spark.openlineage.parentJobNamespace={{ lineage_job_namespace() }} \
--conf spark.openlineage.parentJobName={{ lineage_job_name(task_instance) }} \
--conf spark.openlineage.parentRunId={{ lineage_run_id(task_instance) }} \
--conf spark.openlineage.rootParentJobNamespace={{ lineage_root_job_namespace(task_instance) }} \
--conf spark.openlineage.rootParentJobName={{ lineage_root_job_name(task_instance) }} \
--conf spark.openlineage.rootParentRunId={{ lineage_root_run_id(task_instance) }}\
"
}}'
""",
)

Backport OpenLineage lineage macros for older provider versions

The lineage_root_job_namespace, lineage_root_job_name, and lineage_root_run_id macros that emit the outermost DAG identifiers for root-parent linking were added in apache-airflow-providers-openlineage 2.3.0. If you cannot upgrade the provider (for example, Amazon MWAA on Airflow 2.9.2 pins provider 2.2.0), define the missing macros as user_defined_macros on the DAG:

from __future__ import annotations

from datetime import datetime

from airflow import DAG
from airflow.operators.bash import BashOperator
from airflow.providers.openlineage import conf as ol_conf
from airflow.providers.openlineage.plugins.adapter import OpenLineageAdapter

def _logical_date(ti):
return getattr(ti, "logical_date", None) or ti.execution_date

def _clear_number(ti) -> int:
return ti.get_dagrun().clear_number

def _root_from_conf(ti, key):
ol = (ti.get_dagrun().conf or {}).get("openlineage") or {}
return ol.get(key) or None

def lineage_job_namespace():
return ol_conf.namespace()

def lineage_job_name(ti):
return f"{ti.dag_id}.{ti.task_id}"

def lineage_run_id(ti):
return OpenLineageAdapter.build_task_instance_run_id(
dag_id=ti.dag_id,
task_id=ti.task_id,
try_number=ti.try_number,
logical_date=_logical_date(ti),
map_index=ti.map_index,
)

def lineage_root_job_namespace(ti):
return _root_from_conf(ti, "rootParentJobNamespace") or ol_conf.namespace()

def lineage_root_job_name(ti):
return _root_from_conf(ti, "rootParentJobName") or ti.dag_id

def lineage_root_run_id(ti):
forwarded = _root_from_conf(ti, "rootParentRunId")
if forwarded:
return forwarded
return OpenLineageAdapter.build_dag_run_id(
dag_id=ti.dag_id,
logical_date=_logical_date(ti),
clear_number=_clear_number(ti),
)

OL_MACROS = {
"lineage_job_namespace": lineage_job_namespace,
"lineage_job_name": lineage_job_name,
"lineage_run_id": lineage_run_id,
"lineage_root_job_namespace": lineage_root_job_namespace,
"lineage_root_job_name": lineage_root_job_name,
"lineage_root_run_id": lineage_root_run_id,
}

with DAG(
dag_id="<YOUR_DAG_ID>",
start_date=datetime(2026, 1, 1),
schedule_interval=None,
catchup=False,
user_defined_macros=OL_MACROS,
) as dag:
submit = BashOperator(
task_id="<YOUR_TASK_ID>",
bash_command="...",
)

With user_defined_macros set on the DAG, the {{ lineage_*() }} and {{ lineage_root_*() }} calls in your task templates resolve to values that match the built-in macros shipped with provider 2.3.0+, so downstream Spark or dbt jobs can link to the Airflow root parent in Datadog.

Further Reading

Additional helpful documentation, links, and articles: