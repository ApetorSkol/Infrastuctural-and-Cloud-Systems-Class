What is a Helm Chart?
Helm is a package manager for Kubernetes that simplifies defining, installing, and managing applications. A Helm chart is a collection of files describing Kubernetes resources needed to run an application.

It enables structured packaging, parameterization, and templated deployments to make Kubernetes applications reusable and manageable.

Basic Helm Principles
1. Helm Charts Structure
A typical Helm chart contains:

my-chart/
|-- charts/           # Optional dependencies
|-- templates/        # Templated Kubernetes manifests
|-- values.yaml       # Default configuration values
|-- Chart.yaml        # Chart metadata
|-- README.md         # Documentation
2. Templating in Helm
Helm uses Go templating for dynamic generation. Templates can reference values from values.yaml like so:

apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-config
  namespace: {{ .Values.namespace }}
data:
  APP_ENV: {{ .Values.appEnv | default "production" }}
This lets you customize deployments without changing the templates.

3. Functions in Helm Templating
default: Default value if undefined.
toYaml: Convert dict to YAML.
tpl: Render string templates.
include: Include another template file.
lookup: Query existing resources.
Example:

apiVersion: v1
kind: Secret
metadata:
  name: {{ .Release.Name }}-secret
data:
  username: {{ .Values.secret.username | b64enc }}
  password: {{ .Values.secret.password | b64enc }}
4. Reading Files in Helm
Read a file from the chart with Files.Get:

apiVersion: v1
kind: ConfigMap
metadata:
  name: my-config
data:
  config.yaml: |
    {{ .Files.Get "something.php" | indent 4 }}
5. Chart.yaml in Detail
Defines chart metadata:

apiVersion: v2
name: my-chart
version: 1.0.0
description: A Helm chart for deploying my application
maintainers:
  - name: Developer Name
    email: developer@example.com
appVersion: "1.0.0"
dependencies:
  - name: minio
    version: "5.0.11"
    repository: "https://charts.min.io/"
apiVersion: Chart schema version
name: Chart name
version: Chart version
appVersion: App version
maintainers: Maintainer info
dependencies: Required charts
Assignment
Create a deployment of a simple PHP application with PostgreSQL and Minio. The PostgreSQL will be backed up to the Minio S3 service.

The comments for the most important templates you should have in the chart (you may need more):

Helm chart named pa234-hw3-uco
Templates: Deployment, Ingress, Service, ConfigMap
Apache deployment:
We have prepared container image with all dependencies for PHP script. Use the image:
cerit.io/pa234/php:8.2-apache (should be also templated from values.yaml)
Use runAsUser = 33
Mount PHP file from ConfigMap to /var/www/html/<uco>.php
Mount it as subPath
Inject Minio credentials via environment variables
Make sure the website (the script) runs on one of the paths:
http(s)://host/uco.php or http(s)://host or http(s)://host/index.php
Apache service: 
expose port 80 (you can bind port 80 in this case )
Ingress:
with TLS annotations and host from values.yaml
https://docs.cerit.io/en/docs/kubernetes/expose#web-based-applications
Deploy Minio Tenant via provided Helm chart
Deploy PostgreSQL with CNPG operator and configure S3 backups (https://docs.cerit.io/en/docs/operators/postgres-cnpg)
Secret manifest: shared Minio credentials
Use Minio Tenant subchart for the deployment of Minio:
tenant-7.0.0.tgz
https://github.com/minio/operator/tree/master/helm/tenant

see the values.yml what available to configure.

Here is the PHP script you should use (the script fixed, should work):

<?php
use Aws\S3\S3Client;
   
$AWS_KEY = getenv('AWS_ACCESS_KEY_ID');
$AWS_SECRET_KEY = getenv('AWS_SECRET_ACCESS_KEY');
$ENDPOINT = getenv('AWS_S3_ENDPOINT') ?: 'http://minio';
  
// require the amazon sdk from your composer vendor dir
require __DIR__.'/vendor/autoload.php';
  
// Check if credentials are set
if (!$AWS_KEY || !$AWS_SECRET_KEY) {
    die("AWS credentials are not set in environment variables.\n");
}
  
// Instantiate the S3 class and point it at the desired host
$client = new S3Client([
    'region' =--> 'us-east-1',
    'version' => '2006-03-01',
    'endpoint' => $ENDPOINT,
    'credentials' => [
        'key' => $AWS_KEY,
        'secret' => $AWS_SECRET_KEY
    ],
    // Set the S3 class to use host/bucket
    // instead of bucket.host.
    'use_path_style_endpoint' => true
]);
  
$listResponse = $client->listBuckets();
$buckets = $listResponse['Buckets'];
  
foreach ($buckets as $bucket) {
    echo $bucket['Name'] . "\t" . $bucket['CreationDate'] . "\n";
}   
Here is the values.yml we will be using to test the solution:

tenant:
  tenant:
    name: myminio
    pools:
      - servers: 1
        name: pool-0
        volumesPerServer: 1
        storageClassName: nfs-csi
        size: 10Gi
        resources:
          limits:
            cpu: 1
            memory: 2Gi
        securityContext:
          runAsUser: 1000
          runAsGroup: 1000
          fsGroup: 1000
          fsGroupChangePolicy: "OnRootMismatch"
          runAsNonRoot: true
        containerSecurityContext:
          runAsUser: 1000
          runAsGroup: 1000
          runAsNonRoot: true
          allowPrivilegeEscalation: false
          capabilities:
            drop:
              - ALL
          seccompProfile:
            type: RuntimeDefault
    buckets:
      - name: '<uco>'
    users:
      - name: s3-creds
    certificate:
      requestAutoCert: false
s3:
  access_key: ""
  secret_key: ""
  
host: uco-pa234.dyn.cloud.e-infra.cz
image: cerit.io/pa234/php:8.2-apache
 
db:
  instances: 1
  name: "db-<uco>"
  owner: "<uco>"
  resources:
    requests:
      cpu: 1
      memory: "4Gi"
    limits:
      cpu: 1
      memory: "4Gi"
  storage:
    size: 10Gi
    storageClass: nfs-csi
  backup:
    enable: true
    path: "s3://<uco>/"
    url: "http://minio"


Submission
☠️ Deadline: April 2, 2025 23:59:59
