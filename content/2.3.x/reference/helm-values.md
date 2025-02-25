---
# metadata # 
title:  Helm Chart Values
description: Learn about the configurable helm chart attributes for Pachyderm.
date: 
# taxonomy #
tags: ["configuration", "helm"]
series:
seriesPart:
---


This document discusses each of the fields present in the `values.yaml` that can be used to deploy with Helm.
To see how to use a helm values files to customize your deployment, refer to our [Helm Deployment Documentation](../../deploy-manage/deploy/helm-install/) section.

{{% notice note %}}
You rarely need to specify all the fields. Most fields either come with sensible defaults or can be empty.

Values that are unchanged from the defaults can be omitted from the values file you supply at installation.

Take a look at our deployment instructions [locally](../../getting-started/local-installation/) or [in the cloud](../../deploy-manage/deploy/quickstart/) to identify which of those are required for your deployment target.
{{% /notice %}}

## Values.yaml
The following section displays the complete list of fields available in the [values.yaml](https://github.com/pachyderm/pachyderm/blob/2.3.x/etc/helm/pachyderm/values.yaml). 
Each section is further detailed in its own sub-chapter. 


```yaml
# SPDX-FileCopyrightText: Pachyderm, Inc. <info@pachyderm.com>
# SPDX-License-Identifier: Apache-2.0

# Deploy Target configures the storage backend to use and cloud provider
# settings (storage classes, etc). It must be one of GOOGLE, AMAZON,
# MINIO, MICROSOFT, CUSTOM or LOCAL.
deployTarget: ""

global:
  postgresql:
    # postgresqlUsername is the username to access the pachyderm and dex databases
    postgresqlUsername: "pachyderm"
    # postgresqlPassword to access the postgresql database.  We set a default well-known password to
    # facilitate easy upgrades when testing locally.  Any sort of install that needs to be secure
    # must specify a secure password here, or provide the postgresqlExistingSecretName and
    # postgresqlExistingSecretKey secret.  If using an external Postgres instance (CloudSQL / RDS /
    # etc.), this is the password that Pachyderm will use to connect to it.
    postgresqlPassword: "insecure-user-password"
    # When installing a local Postgres instance, postgresqlPostgresPassword defines the root
    # ('postgres') user's password.  It must remain consistent between upgrades, and must be
    # explicitly set to a value if security is desired.  Pachyderm does not use this account; this
    # password is only required so that administrators can manually perform administrative tasks.
    postgresqlPostgresPassword: "insecure-root-password"
    # If you want to supply the postgresql password in an existing secret, leave Password blank and
    # Supply the name of the existing secret in the namespace and the key in that secret with the password
    postgresqlExistingSecretName: ""
    postgresqlExistingSecretKey: ""
    # postgresqlDatabase is the database name where pachyderm data will be stored
    postgresqlDatabase: "pachyderm"
    # The postgresql database host to connect to. Defaults to postgres service in subchart
    postgresqlHost: "postgres"
    # The postgresql database port to connect to. Defaults to postgres server in subchart
    postgresqlPort: "5432"
    # postgresqlSSL is the SSL mode to use for pg-bouncer connecting to Postgres, for the default local postgres it is disabled
    postgresqlSSL: "disable"
    # CA Certificate required to connect to Postgres
    postgresqlSSLCACert: ""
    # TLS Secret with cert/key to connect to Postgres
    postgresqlSSLSecret: ""
    # Indicates the DB name that dex connects to
    # Indicates the DB name that dex connects to. Defaults to "Dex" if not set.
    identityDatabaseFullNameOverride: ""
  # imagePullSecrets allow you to pull images from private repositories, these will also be added to pipeline workers
  # https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/
  # Example:
  # imagePullSecrets:
  #   - regcred
  imagePullSecrets: []
  # when set, the certificate file in pachd-tls-cert will be loaded as the root certificate for pachd, console, and enterprise-server pods
  customCaCerts: false
  # Sets the HTTP/S proxy server address for console, pachd, and enterprise server.  (This is for
  # traffic leaving the cluster, not traffic coming into the cluster.)
  proxy: ""
  # If proxy is set, this allows you to set a comma-separated list of destinations that bypass the proxy
  noProxy: ""
  # Set security context runAs users. If running on openshift, set enabled to false as openshift creates its own contexts.
  securityContexts:
    enabled: true

console:
  # enabled controls whether the console manifests are created or not.
  enabled: true
  annotations: {}
  image:
    # repository is the image repo to pull from; together with tag it
    # replicates the --console-image & --registry arguments to pachctl
    # deploy.
    repository: "pachyderm/haberdashery"
    pullPolicy: "IfNotPresent"
    # tag is the image repo to pull from; together with repository it
    # replicates the --console-image argument to pachctl deploy.
    tag: "2.3.3-1"
  priorityClassName: ""
  nodeSelector: {}
  tolerations: []
  # podLabels specifies labels to add to the console pod.
  podLabels: {}
  # resources specifies the resource request and limits.
  resources:
    {}
    #limits:
    #  cpu: "1"
    #  memory: "2G"
    #requests:
    #  cpu: "1"
    #  memory: "2G"
  config:
    reactAppRuntimeIssuerURI: "" # Inferred if running locally or using ingress
    oauthRedirectURI: "" # Infered if running locally or using ingress
    oauthClientID: "console"
    oauthClientSecret: "" # Autogenerated on install if blank
    # oauthClientSecretSecretName is used to set the OAuth Client Secret via an existing k8s secret.
    # The value is pulled from the key, "OAUTH_CLIENT_SECRET".
    oauthClientSecretSecretName: ""
    graphqlPort: 4000
    pachdAddress: "pachd-peer:30653"
    disableTelemetry: false # Disables analytics and error data collection

  service:
    annotations: {}
    # labels specifies labels to add to the console service.
    labels: {}
    # type specifies the Kubernetes type of the console service.
    type: ClusterIP

etcd:
  affinity: {}
  annotations: {}
  # dynamicNodes sets the number of nodes in the etcd StatefulSet.  It
  # is analogous to the --dynamic-etcd-nodes argument to pachctl
  # deploy.
  dynamicNodes: 1
  image:
    repository: "pachyderm/etcd"
    tag: "v3.5.1"
    pullPolicy: "IfNotPresent"
  # maxTxnOps sets the --max-txn-ops in the container args
  maxTxnOps: 10000
  priorityClassName: ""
  nodeSelector: {}
  # podLabels specifies labels to add to the etcd pod.
  podLabels: {}
  # resources specifies the resource request and limits
  resources:
    {}
    #limits:
    #  cpu: "1"
    #  memory: "2G"
    #requests:
    #  cpu: "1"
    #  memory: "2G"
  # storageClass indicates the etcd should use an existing
  # StorageClass for its storage.  It is analogous to the
  # --etcd-storage-class argument to pachctl deploy.
  # More info for setting up storage classes on various cloud providers:
  # AWS: https://docs.aws.amazon.com/eks/latest/userguide/storage-classes.html
  # GCP: https://cloud.google.com/compute/docs/disks/performance#disk_types
  # Azure: https://docs.microsoft.com/en-us/azure/aks/concepts-storage#storage-classes
  storageClass: ""
  # storageSize specifies the size of the volume to use for etcd.
  # Recommended Minimum Disk size for Microsoft/Azure: 256Gi  - 1,100 IOPS https://azure.microsoft.com/en-us/pricing/details/managed-disks/
  # Recommended Minimum Disk size for Google/GCP: 50Gi        - 1,500 IOPS https://cloud.google.com/compute/docs/disks/performance
  # Recommended Minimum Disk size for Amazon/AWS: 500Gi (GP2) - 1,500 IOPS https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-volume-types.html
  storageSize: 10Gi
  service:
    # annotations specifies annotations to add to the etcd service.
    annotations: {}
    # labels specifies labels to add to the etcd service.
    labels: {}
    # type specifies the Kubernetes type of the etcd service.
    type: ClusterIP
  tolerations: []

enterpriseServer:
  enabled: false
  affinity: {}
  annotations: {}
  tolerations: []
  priorityClassName: ""
  nodeSelector: {}
  service:
    type: ClusterIP
    apiGRPCPort: 31650
    prometheusPort: 31656
    oidcPort: 31657
    identityPort: 31658
    s3GatewayPort: 31600
  # There are three options for TLS:
  # 1. Disabled
  # 2. Enabled, existingSecret, specify secret name
  # 3. Enabled, newSecret, must specify cert, key and name
  tls:
    enabled: false
    secretName: ""
    newSecret:
      create: false
      crt: ""
      key: ""
  resources:
    {}
    #limits:
    #  cpu: "1"
    #  memory: "2G"
    #requests:
    #  cpu: "1"
    #  memory: "2G"
  # podLabels specifies labels to add to the pachd pod.
  podLabels: {}
  clusterDeploymentID: ""
  image:
    repository: "pachyderm/pachd"
    pullPolicy: "IfNotPresent"
    # tag defaults to the chart’s specified appVersion.
    tag: ""

ingress:
  enabled: false
  annotations: {}
  host: ""
  # when set to true, uriHttpsProtoOverride will add the https protocol to the ingress URI routes without configuring certs
  uriHttpsProtoOverride: false
  # There are three options for TLS:
  # 1. Disabled
  # 2. Enabled, existingSecret, specify secret name
  # 3. Enabled, newSecret, must specify cert, key, secretName and set newSecret.create to true
  tls:
    enabled: false
    secretName: ""
    newSecret:
      create: false
      crt: ""
      key: ""

# loki-stack contains values that will be passed to the loki-stack subchart
loki-stack:
  loki:
    serviceAccount:
      automountServiceAccountToken: false
    persistence:
      enabled: true
      accessModes:
        - ReadWriteOnce
      size: 10Gi
      # More info for setting up storage classes on various cloud providers:
      # AWS: https://docs.aws.amazon.com/eks/latest/userguide/storage-classes.html
      # GCP: https://cloud.google.com/compute/docs/disks/performance#disk_types
      # Azure: https://docs.microsoft.com/en-us/azure/aks/concepts-storage#storage-classes
      storageClassName: ""
      annotations: {}
      priorityClassName: ""
      nodeSelector: {}
      tolerations: []
    config:
      limits_config:
        retention_period: 24h
        retention_stream:
          - selector: '{suite="pachyderm"}'
            priority: 1
            period: 168h # = 1 week
  grafana:
    enabled: false
  promtail:
    config:
      clients:
        - url: "http://{{ .Release.Name }}-loki:3100/loki/api/v1/push"
      snippets:
        # The scrapeConfigs section is copied from loki-stack-2.6.4
        # The pipeline_stages.match stanza has been added to prevent multiple lokis in a cluster from mixing their logs.
        scrapeConfigs: |
          - job_name: kubernetes-pods
            pipeline_stages:
              {{- toYaml .Values.config.snippets.pipelineStages | nindent 4 }}
              - match:
                  selector: '{namespace!="{{ .Release.Namespace }}"}'
                  action: drop
            kubernetes_sd_configs:
              - role: pod
            relabel_configs:
              - source_labels:
                  - __meta_kubernetes_pod_controller_name
                regex: ([0-9a-z-.]+?)(-[0-9a-f]{8,10})?
                action: replace
                target_label: __tmp_controller_name
              - source_labels:
                  - __meta_kubernetes_pod_label_app_kubernetes_io_name
                  - __meta_kubernetes_pod_label_app
                  - __tmp_controller_name
                  - __meta_kubernetes_pod_name
                regex: ^;*([^;]+)(;.*)?$
                action: replace
                target_label: app
              - source_labels:
                  - __meta_kubernetes_pod_label_app_kubernetes_io_instance
                  - __meta_kubernetes_pod_label_release
                regex: ^;*([^;]+)(;.*)?$
                action: replace
                target_label: instance
              - source_labels:
                  - __meta_kubernetes_pod_label_app_kubernetes_io_component
                  - __meta_kubernetes_pod_label_component
                regex: ^;*([^;]+)(;.*)?$
                action: replace
                target_label: component
              {{- if .Values.config.snippets.addScrapeJobLabel }}
              - replacement: kubernetes-pods
                target_label: scrape_job
              {{- end }}
              {{- toYaml .Values.config.snippets.common | nindent 4 }}
              {{- with .Values.config.snippets.extraRelabelConfigs }}
              {{- toYaml . | nindent 4 }}
              {{- end }}
        pipelineStages:
          - cri: {}
        common:
          # This is copy and paste of existing actions, so we don't lose them.
          # Cf. https://github.com/grafana/loki/issues/3519#issuecomment-1125998705
          - action: replace
            source_labels:
              - __meta_kubernetes_pod_node_name
            target_label: node_name
          - action: replace
            source_labels:
              - __meta_kubernetes_namespace
            target_label: namespace
          - action: replace
            replacement: $1
            separator: /
            source_labels:
              - namespace
              - app
            target_label: job
          - action: replace
            source_labels:
              - __meta_kubernetes_pod_name
            target_label: pod
          - action: replace
            source_labels:
              - __meta_kubernetes_pod_container_name
            target_label: container
          - action: replace
            replacement: /var/log/pods/*$1/*.log
            separator: /
            source_labels:
              - __meta_kubernetes_pod_uid
              - __meta_kubernetes_pod_container_name
            target_label: __path__
          - action: replace
            regex: true/(.*)
            replacement: /var/log/pods/*$1/*.log
            separator: /
            source_labels:
              - __meta_kubernetes_pod_annotationpresent_kubernetes_io_config_hash
              - __meta_kubernetes_pod_annotation_kubernetes_io_config_hash
              - __meta_kubernetes_pod_container_name
            target_label: __path__
          - action: keep
            regex: pachyderm
            source_labels:
              - __meta_kubernetes_pod_label_suite
          # this gets all kubernetes labels as well
          - action: labelmap
            regex: __meta_kubernetes_pod_label_(.+)
    livenessProbe:
      failureThreshold: 5
      tcpSocket:
        port: http-metrics
      initialDelaySeconds: 10
      periodSeconds: 10
      successThreshold: 1
      timeoutSeconds: 1

pachd:
  enabled: true
  preflightChecks:
    # if enabled runs kube validation preflight checks.
    enabled: true
  affinity: {}
  annotations: {}
  # clusterDeploymentID sets the Pachyderm cluster ID.
  clusterDeploymentID: ""
  configJob:
    annotations: {}
  # goMaxProcs is passed as GOMAXPROCS to the pachd container.
  goMaxProcs: 0
  image:
    repository: "pachyderm/pachd"
    pullPolicy: "IfNotPresent"
    # tag defaults to the chart’s specified appVersion.
    # This sets the worker image tag as well (they should be kept in lock step)
    tag: ""
  logFormat: "json"
  logLevel: "info"
  # If lokiDeploy is true, a Pachyderm-specific instance of Loki will
  # be deployed.
  lokiDeploy: true
  # lokiLogging enables Loki logging if set.
  lokiLogging: true
  metrics:
    # enabled sets the METRICS environment variable if set.
    enabled: true
    # endpoint should be the URL of the metrics endpoint.
    endpoint: ""
  priorityClassName: ""
  nodeSelector: {}
  # podLabels specifies labels to add to the pachd pod.
  podLabels: {}
  # resources specifies the resource requests and limits
  # replicas sets the number of pachd running pods
  replicas: 1
  resources:
    {}
    #limits:
    #  cpu: "1"
    #  memory: "2G"
    #requests:
    #  cpu: "1"
    #  memory: "2G"
  # requireCriticalServersOnly only requires the critical pachd
  # servers to startup and run without errors.  It is analogous to the
  # --require-critical-servers-only argument to pachctl deploy.
  requireCriticalServersOnly: false
  # If enabled, External service creates a service which is safe to
  # be exposed externally
  externalService:
    enabled: false
    # (Optional) specify the existing IP Address of the load balancer
    loadBalancerIP: ""
    apiGRPCPort: 30650
    s3GatewayPort: 30600
    annotations: {}
  service:
    # labels specifies labels to add to the pachd service.
    labels: {}
    # type specifies the Kubernetes type of the pachd service.
    type: "ClusterIP"
    annotations: {}
    apiGRPCPort: 30650
    prometheusPort: 30656
    oidcPort: 30657
    identityPort: 30658
    s3GatewayPort: 30600
    #apiGrpcPort:
    #  expose: true
    #  port: 30650
  # DEPRECATED: activateEnterprise is no longer used.
  activateEnterprise: false
  ## if pachd.activateEnterpriseMember is set, enterprise will be activated and connected to an existing enterprise server.
  ## if pachd.enterpriseLicenseKey is set, enterprise will be activated.
  activateEnterpriseMember: false
  ## if pachd.activateAuth is set, auth will be bootstrapped by the config-job.
  activateAuth: true
  ## the license key used to activate enterprise features
  enterpriseLicenseKey: ""
  # enterpriseLicenseKeySecretName is used to pass the enterprise license key value via an existing k8s secret.
  # The value is pulled from the key, "enterprise-license-key".
  enterpriseLicenseKeySecretName: ""
  # if a token is not provided, a secret will be autogenerated on install and stored in the k8s secret 'pachyderm-bootstrap-config.rootToken'
  rootToken: ""
  # rootTokenSecretName is used to pass the rootToken value via an existing k8s secret
  # The value is pulled from the key, "root-token".
  rootTokenSecretName: ""
  # if a secret is not provided, a secret will be autogenerated on install and stored in the k8s secret 'pachyderm-bootstrap-config.enterpriseSecret'
  enterpriseSecret: ""
  # enterpriseSecretSecretName is used to pass the enterprise secret value via an existing k8s secret.
  # The value is pulled from the key, "enterprise-secret".
  enterpriseSecretSecretName: ""
  # if a secret is not provided, a secret will be autogenerated on install and stored in the k8s secret 'pachyderm-bootstrap-config.authConfig.clientSecret'
  oauthClientID: pachd
  oauthClientSecret: ""
  # oauthClientSecretSecretName is used to set the OAuth Client Secret via an existing k8s secret.
  # The value is pulled from the key, "pachd-oauth-client-secret".
  oauthClientSecretSecretName: ""
  oauthRedirectURI: ""
  # DEPRECATED: enterpriseRootToken is deprecated, in favor of enterpriseServerToken
  # NOTE only used if pachd.activateEnterpriseMember == true
  enterpriseRootToken: ""
  # DEPRECATED: enterpriseRootTokenSecretName is deprecated in favor of enterpriseServerTokenSecretName
  # enterpriseRootTokenSecretName is used to pass the enterpriseRootToken value via an existing k8s secret.
  # The value is pulled from the key, "enterprise-root-token".
  enterpriseRootTokenSecretName: ""
  # enterpriseServerToken represents a token that can authenticate to a separate pachyderm enterprise server,
  # and is used to complete the enterprise member registration process for this pachyderm cluster.
  # The user backing this token should have either the licenseAdmin & identityAdmin roles assigned, or
  # the clusterAdmin role.
  # NOTE: only used if pachd.activateEnterpriseMember == true
  enterpriseServerToken: ""
  # enterpriseServerTokenSecretName is used to pass the enterpriseServerToken value via an existing k8s secret.
  # The value is pulled from the key, "enterprise-server-token".
  enterpriseServerTokenSecretName: ""
  # only used if pachd.activateEnterpriseMember == true
  enterpriseServerAddress: ""
  enterpriseCallbackAddress: ""
  # Indicates to pachd whether dex is embedded in its process.
  localhostIssuer: "" # "true", "false", or "" (used string as bool doesn't support empty value)
  # set the initial pachyderm cluster role bindings, mapping a user to their list of roles
  # ex.
  # pachAuthClusterRoleBindings:
  #   robot:wallie:
  #   - repoReader
  #   robot:eve:
  #   - repoWriter
  pachAuthClusterRoleBindings: {}
  # additionalTrustedPeers is used to configure the identity service to recognize additional OIDC clients as trusted peers of pachd.
  # For example, see the following example or the dex docs (https://dexidp.io/docs/custom-scopes-claims-clients/#cross-client-trust-and-authorized-party).
  # additionalTrustedPeers:
  #   - example-app
  additionalTrustedPeers: []
  serviceAccount:
    create: true
    additionalAnnotations: {}
    name: "pachyderm" #TODO Set default in helpers / Wire up in templates
  storage:
    # backend configures the storage backend to use.  It must be one
    # of GOOGLE, AMAZON, MINIO, MICROSOFT or LOCAL. This is set automatically
    # if deployTarget is GOOGLE, AMAZON, MICROSOFT, or LOCAL
    backend: ""
    amazon:
      # bucket sets the S3 bucket to use.
      bucket: ""
      # cloudFrontDistribution sets the CloudFront distribution in the
      # storage secrets.  It is analogous to the
      # --cloudfront-distribution argument to pachctl deploy.
      cloudFrontDistribution: ""
      customEndpoint: ""
      # disableSSL disables SSL.  It is analogous to the --disable-ssl
      # argument to pachctl deploy.
      disableSSL: false
      # id sets the Amazon access key ID to use.  Together with secret
      # and token, it implements the functionality of the
      # --credentials argument to pachctl deploy.
      id: ""
      # logOptions sets various log options in Pachyderm’s internal S3
      # client.  Comma-separated list containing zero or more of:
      # 'Debug', 'Signing', 'HTTPBody', 'RequestRetries',
      # 'RequestErrors', 'EventStreamBody', or 'all'
      # (case-insensitive).  See 'AWS SDK for Go' docs for details.
      # logOptions is analogous to the --obj-log-options argument to
      # pachctl deploy.
      logOptions: ""
      # maxUploadParts sets the maximum number of upload parts.  It is
      # analogous to the --max-upload-parts argument to pachctl
      # deploy.
      maxUploadParts: 10000
      # verifySSL performs SSL certificate verification.  It is the
      # inverse of the --no-verify-ssl argument to pachctl deploy.
      verifySSL: true
      # partSize sets the part size for object storage uploads.  It is
      # analogous to the --part-size argument to pachctl deploy.  It
      # has to be a string due to Helm and YAML parsing integers as
      # floats.  Cf. https://github.com/helm/helm/issues/1707
      partSize: "5242880"
      # region sets the AWS region to use.
      region: ""
      # retries sets the number of retries for object storage
      # requests.  It is analogous to the --retries argument to
      # pachctl deploy.
      retries: 10
      # reverse reverses object storage paths.  It is analogous to the
      # --reverse argument to pachctl deploy.
      reverse: true
      # secret sets the Amazon secret access key to use.  Together with id
      # and token, it implements the functionality of the
      # --credentials argument to pachctl deploy.
      secret: ""
      # timeout sets the timeout for object storage requests.  It is
      # analogous to the --timeout argument to pachctl deploy.
      timeout: "5m"
      # token optionally sets the Amazon token to use.  Together with
      # id and secret, it implements the functionality of the
      # --credentials argument to pachctl deploy.
      token: ""
      # uploadACL sets the upload ACL for object storage uploads.  It
      # is analogous to the --upload-acl argument to pachctl deploy.
      uploadACL: "bucket-owner-full-control"
    google:
      bucket: ""
      # cred is a string containing a GCP service account private key,
      # in object (JSON or YAML) form.  A simple way to pass this on
      # the command line is with the set-file flag, e.g.:
      #
      #  helm install pachd -f my-values.yaml --set-file storage.google.cred=creds.json pachyderm/pachyderm
      cred: ""
      # Example:
      # cred: |
      #  {
      #    "type": "service_account",
      #    "project_id": "…",
      #    "private_key_id": "…",
      #    "private_key": "-----BEGIN PRIVATE KEY-----\n…\n-----END PRIVATE KEY-----\n",
      #    "client_email": "…@….iam.gserviceaccount.com",
      #    "client_id": "…",
      #    "auth_uri": "https://accounts.google.com/o/oauth2/auth",
      #    "token_uri": "https://oauth2.googleapis.com/token",
      #    "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
      #    "client_x509_cert_url": "https://www.googleapis.com/robot/v1/metadata/x509/…%40….iam.gserviceaccount.com"
      #  }
    local:
      # hostPath indicates the path on the host where the PFS metadata
      # will be stored.  It must end in /.  It is analogous to the
      # --host-path argument to pachctl deploy.
      hostPath: ""
      requireRoot: true #Root required for hostpath, but we run rootless in CI
    microsoft:
      container: ""
      id: ""
      secret: ""
    minio:
      # minio bucket name
      bucket: ""
      # the minio endpoint. Should only be the hostname:port, no http/https.
      endpoint: ""
      # the username/id with readwrite access to the bucket.
      id: ""
      # the secret/password of the user with readwrite access to the bucket.
      secret: ""
      # enable https for minio with "true" defaults to "false"
      secure: ""
      # Enable S3v2 support by setting signature to "1". This feature is being deprecated
      signature: ""
    # putFileConcurrencyLimit sets the maximum number of files to
    # upload or fetch from remote sources (HTTP, blob storage) using
    # PutFile concurrently.  It is analogous to the
    # --put-file-concurrency-limit argument to pachctl deploy.
    putFileConcurrencyLimit: 100
    # uploadConcurrencyLimit sets the maximum number of concurrent
    # object storage uploads per Pachd instance.  It is analogous to
    # the --upload-concurrency-limit argument to pachctl deploy.
    uploadConcurrencyLimit: 100
    # The shard size corresponds to the total size of the files in a shard.
    # The shard count corresponds to the total number of files in a shard.
    # If either criteria is met, a shard will be created.
    compactionShardSizeThreshold: 0
    compactionShardCountThreshold: 0
  ppsWorkerGRPCPort: 1080
  # the number of seconds between pfs's garbage collection cycles.
  # if this value is set to 0, it will default to pachyderm's internal configuration.
  # if this value is less than 0, it will turn off garbage collection.
  storageGCPeriod: 0
  # the number of seconds between chunk garbage colletion cycles.
  # if this value is set to 0, it will default to pachyderm's internal configuration.
  # if this value is less than 0, it will turn off chunk garbage collection.
  storageChunkGCPeriod: 0
  # There are three options for TLS:
  # 1. Disabled
  # 2. Enabled, existingSecret, specify secret name
  # 3. Enabled, newSecret, must specify cert, key and name
  tls:
    enabled: false
    secretName: ""
    newSecret:
      create: false
      crt: ""
      key: ""
  tolerations: []
  worker:
    image:
      repository: "pachyderm/worker"
      pullPolicy: "IfNotPresent"
      # Worker tag is set under pachd.image.tag (they should be kept in lock step)
    serviceAccount:
      create: true
      additionalAnnotations: {}
      # name sets the name of the worker service account.  Analogous to
      # the --worker-service-account argument to pachctl deploy.
      name: "pachyderm-worker" #TODO Set default in helpers / Wire up in templates
  rbac:
    # create indicates whether RBAC resources should be created.
    # Setting it to false is analogous to passing --no-rbac to pachctl
    # deploy.
    create: true

kubeEventTail:
  # Deploys a lightweight app that watches kubernetes events and echos them to logs.
  enabled: true
  # clusterScope determines whether kube-event-tail should watch all events or just events in its namespace.
  clusterScope: false
  image:
    repository: pachyderm/kube-event-tail
    pullPolicy: "IfNotPresent"
    tag: "v0.0.6"
  resources:
    limits:
      cpu: "1"
      memory: 100Mi
    requests:
      cpu: 100m
      memory: 45Mi

pgbouncer:
  service:
    type: ClusterIP
  annotations: {}
  priorityClassName: ""
  nodeSelector: {}
  tolerations: []
  image:
    repository: pachyderm/pgbouncer
    tag: 1.16.1-debian-10-r82
  resources:
    {}
    #limits:
    #  cpu: "1"
    #  memory: "2G"
    #requests:
    #  cpu: "1"
    #  memory: "2G"
  # maxConnections specifies the maximum number of concurrent connections into pgbouncer.
  maxConnections: 1000
  # defaultPoolSize specifies the maximum number of concurrent connections from pgbouncer to the postgresql database.
  defaultPoolSize: 20

# Note: Postgres values control the Bitnami Postgresql Subchart
postgresql:
  # enabled controls whether to install postgres or not.
  # If not using the built in Postgres, you must specify a Postgresql
  # database server to connect to in global.postgresql
  # The enabled value is watched by the 'condition' set on the Postgresql
  # dependency in Chart.yaml
  enabled: true
  image:
    tag: "13.3.0"
  # DEPRECATED from pachyderm 2.1.5
  initdbScripts:
    dex.sh: |
      #!/bin/bash
      set -e
      psql -v ON_ERROR_STOP=1 --username "$POSTGRES_USER" --dbname "$POSTGRES_DB" <<-EOSQL
        CREATE DATABASE dex;
        GRANT ALL PRIVILEGES ON DATABASE dex TO "$POSTGRES_USER";
      EOSQL
  fullnameOverride: postgres
  persistence:
    # Specify the storage class for the postgresql Persistent Volume (PV)
    # See notes in Bitnami chart values.yaml file for more information.
    # More info for setting up storage classes on various cloud providers:
    # AWS: https://docs.aws.amazon.com/eks/latest/userguide/storage-classes.html
    # GCP: https://cloud.google.com/compute/docs/disks/performance#disk_types
    # Azure: https://docs.microsoft.com/en-us/azure/aks/concepts-storage#storage-classes
    storageClass: ""
    # storageSize specifies the size of the volume to use for postgresql
    # Recommended Minimum Disk size for Microsoft/Azure: 256Gi  - 1,100 IOPS https://azure.microsoft.com/en-us/pricing/details/managed-disks/
    # Recommended Minimum Disk size for Google/GCP: 50Gi        - 1,500 IOPS https://cloud.google.com/compute/docs/disks/performance
    # Recommended Minimum Disk size for Amazon/AWS: 500Gi (GP2) - 1,500 IOPS https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-volume-types.html
    size: 10Gi
    labels:
      suite: pachyderm
  primary:
    priorityClassName: ""
    nodeSelector: {}
    tolerations: []
  readReplicas:
    priorityClassName: ""
    nodeSelector: {}
    tolerations: []

cloudsqlAuthProxy:
  # connectionName may be found by running `gcloud sql instances describe INSTANCE_NAME --project PROJECT_ID`
  connectionName: ""
  serviceAccount: ""
  iamLogin: false
  port: 5432
  enabled: false
  image:
    # repository is the image repo to pull from; together with tag it
    # replicates the --dash-image & --registry arguments to pachctl
    # deploy.
    repository: "gcr.io/cloudsql-docker/gce-proxy"
    pullPolicy: "IfNotPresent"
    # tag is the image repo to pull from; together with repository it
    # replicates the --dash-image argument to pachctl deploy.
    tag: "1.23.0"
  priorityClassName: ""
  nodeSelector: {}
  tolerations: []
  # podLabels specifies labels to add to the dash pod.
  podLabels: {}
  # resources specifies the resource request and limits.
  resources: {}
  #  requests:
  #    # The proxy's memory use scales linearly with the number of active
  #    # connections. Fewer open connections will use less memory. Adjust
  #    # this value based on your application's requirements.
  #    memory: ""
  #    # The proxy's CPU use scales linearly with the amount of IO between
  #    # the database and the application. Adjust this value based on your
  #    # application's requirements.
  #    cpu: ""
  service:
    # labels specifies labels to add to the cloudsql auth proxy service.
    labels: {}
    # type specifies the Kubernetes type of the cloudsql auth proxy service.
    type: ClusterIP

oidc:
  issuerURI: "" #Inferred if running locally or using ingress
  requireVerifiedEmail: false
  # IDTokenExpiry is parsed into golang's time.Duration: https://pkg.go.dev/time#example-ParseDuration
  IDTokenExpiry: 24h
  # (Optional) If set, enables OIDC rotation tokens, and specifies the duration where they are valid.
  # RotationTokenExpiry is parsed into golang's time.Duration: https://pkg.go.dev/time#example-ParseDuration
  RotationTokenExpiry: 48h
  # (Optional) Only set in cases where the issuerURI is not user accessible (ie. localhost install)
  userAccessibleOauthIssuerHost: ""
  ## to set up upstream IDPs, set pachd.mockIDP to false,
  ## and populate the pachd.upstreamIDPs with an array of Dex Connector configurations.
  ## See the example below or https://dexidp.io/docs/connectors/
  # upstreamIDPs:
  #   - id: idpConnector
  #     config:
  #       issuer: ""
  #       clientID: ""
  #       clientSecret: ""
  #       redirectURI: "http://localhost:30658/callback"
  #       insecureEnableGroups: true
  #       insecureSkipEmailVerified: true
  #       insecureSkipIssuerCallbackDomainCheck: true
  #       forwardedLoginParams:
  #       - login_hint
  #     name: idpConnector
  #     type: oidc
  #
  #   - id: okta
  #     config:
  #       issuer: "https://dev-84362674.okta.com"
  #       clientID: "client_id"
  #       clientSecret: "notsecret"
  #       redirectURI: "http://localhost:30658/callback"
  #       insecureEnableGroups: true
  #       insecureSkipEmailVerified: true
  #       insecureSkipIssuerCallbackDomainCheck: true
  #       forwardedLoginParams:
  #       - login_hint
  #     name: okta
  #     type: oidc
  upstreamIDPs: []
  # upstreamIDPsSecretName is used to pass the upstreamIDPs value via an existing k8s secret.
  # The value is pulled from the secret key, "upstream-idps".
  upstreamIDPsSecretName: ""
  # Some dex configurations (like Google) require a credential file. Whatever secret is included in this
  # below secret will be mounted to the pachd pod at /dexcreds/ so for example serviceAccountFilePath: /dexcreds/googleAuth.json
  dexCredentialSecretName: ""
  mockIDP: true
  # additionalClients specifies a list of clients for the cluster to recognize
  # See the ecample below or the dex docs (https://dexidp.io/docs/using-dex/#configuring-your-app).
  # additionalOIDCClient:
  #   - id: example-app
  #     secret: example-app-secret
  #     name: 'Example App'
  #     redirectURIs:
  #     - 'http://127.0.0.1:5555/callback'
  additionalClients: []
  additionalClientsSecretName: ""
  #TODO scopes:

testConnection:
  image:
    repository: alpine
    tag: latest

# The proxy is a service to handle all Pachyderm traffic (S3, Console, OIDC, Dex, GRPC) on a single
# port; good for exposing directly to the Internet.
proxy:
  # If enabled, create a proxy deployment (based on the Envoy proxy) and a service to expose it.  If
  # ingress is also enabled, any Ingress traffic will be routed through the proxy before being sent
  # to pachd or Console.
  enabled: false
  # The external hostname (including port if nonstandard) that the proxy will be reachable at.
  # If you have ingress enabled and an ingress hostname defined, the proxy will use that.
  # Ingress will be deprecated in the future so configuring the proxy host instead is recommended.
  host: ""
  # The number of proxy replicas to run.  1 should be fine, but if you want more for higher
  # availability, that's perfectly reasonable.  Each replica can handle 50,000 concurrent
  # connections.  There is an affinity rule to prefer scheduling the proxy pods on the same node as
  # pachd, so a number here that matches the number of pachd replicas is a fine configuration.
  # (Note that we don't guarantee to keep the proxy<->pachd traffic on-node or even in-region.)
  replicas: 1
  # The envoy image to pull.
  image:
    repository: "envoyproxy/envoy"
    tag: "v1.22.0"
    pullPolicy: "IfNotPresent"
  # Set up resources.  The proxy is configured to shed traffic before using 500MB of RAM, so that's
  # a resonable memory limit.  It doesn't need much CPU.
  resources:
    requests:
      cpu: 100m
      memory: 512Mi
    limits:
      memory: 512Mi
  # Any additional labels to add to the pods.  These are also added to the deployment and service
  # selectors.
  labels: {}
  # Any additional annotations to add to the pods.
  annotations: {}
  # Configure the service that routes traffic to the proxy.
  service:
    # The type of service can be ClusterIP, NodePort, or LoadBalancer.
    type: ClusterIP
    # If the service is a LoadBalancer, you can specify the IP address to use.
    loadBalancerIP: ""
    # The port to serve plain HTTP traffic on.
    httpPort: 80
    # The port to serve HTTPS traffic on, if enabled below.
    httpsPort: 443
    # If the service is a NodePort, you can specify the port to receive HTTP traffic on.
    httpNodePort: 30080
    httpsNodePort: 30443
    # Any additional annotations to add.
    annotations: {}
    # Any additional labels to add to the service itself (not the selector!).
    labels: {}
    # The proxy can also serve each backend service on a numbered port, and will do so for any port
    # not numbered 0 here.  If this service is of type NodePort, the port numbers here will be used
    # for the node port, and will need to be in the node port range.
    legacyPorts:
      console: 0 # legacy 30080, conflicts with default httpNodePort
      grpc: 0 # legacy 30650
      s3Gateway: 0 # legacy 30600
      oidc: 0 # legacy 30657
      identity: 0 # legacy 30658
      metrics: 0 # legacy 30656
  # Configuration for TLS (SSL, HTTPS).
  tls:
    # If true, enable TLS serving.  Enabling TLS is incompatible with support for legacy ports (you
    # can't get a generally-trusted certificate for port numbers), and disables support for
    # cleartext communication (cleartext requests will redirect to the secure server, and HSTS
    # headers are set to prevent downgrade attacks).
    #
    # Note that if you are planning on putting the proxy behind an ingress controller, you probably
    # want to configure TLS for the ingress controller, not the proxy.  This is intended for the
    # case where the proxy is exposed directly to the Internet.  (It is possible to have your
    # ingress controller talk to the proxy over TLS, in which case, it's fine to enable TLS here in
    # addition to in the ingress section above.)
    enabled: false
    # The secret containing "tls.key" and "tls.crt" keys that contain PEM-encoded private key and
    # certificate material.  Generate one with "kubectl create secret tls <name> --key=tls.key
    # --cert=tls.cert".  This format is compatible with the secrets produced by cert-manager, and
    # the proxy will pick up new data when cert-manager rotates the certificate.
    secretName: ""
    # If set, generate the secret from values here.  This is intended only for unit tests.
    secret: {}
```
### deployTarget

`deployTarget` is where you're deploying pachyderm. It configures the storage backend to use and cloud provider settings.
It must be one of:

- `GOOGLE`
- `AMAZON`
- `MINIO`
- `MICROSOFT`
- `CUSTOM`
- `LOCAL`

### global

#### global.postgreSQL

This section is to configure the connection to the postgresql database. By default, it uses the included postgres service.

- `postgresqlUsername` is the username to access the pachyderm and dex databases
- `postgresqlPassword` to access the postgresql database. If blank, a value will be generated by the postgres subchart
-  `postgresqlExistingSecretKey` the secret name of the secret containing `postgresqlPassword` (key: `postgresql-postgres-password`)
When using autogenerated value for the initial install, it must be pulled from the postgres secret and added to values.yaml for future helm upgrades.
- `postgresqlDatabase` is the database name where pachyderm data will be stored
- `postgresqlHost` is the postgresql database host to connect to.
- `postgresqlPort` is the postgresql database port to connect to.
- `postgresqlSSL` is the SSL mode to use for connecting to Postgres, for the default local postgres it is disabled

#### global.imagePullSecrets

`imagePullSecrets` allow you to pull images from private repositories, these will also be added to pipeline workers https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/

Example:
```yaml
  imagePullSecrets:
    - regcred
```
### console

This section is to configure the Pachyderm UI (`console`). It is enabled by default.

- `console.enabled` turns on the deployment of the UI.

- `console.image` sets the image to use for the console. This can be left at the defaults unless instructed.

- `console.podLabels` specifies labels to add to the console pod.

- `console.resources` specifies resources and limits in standard kubernetes format. It is left unset by default.

- `console.service.labels` specifies labels to add to the console service.

- `console.service.type` specifies the Kubernetes type of the console service. The default is `ClusterIP`.

#### console.config

This is where the primary configuration settings for the console are configured, including authentication.

- `config.reactAppRuntimeIssuerURI` this is the pachd oauth address thats accesible to clients outside of the cluster itself. When running local with `kubectl port-forward` this would be set to localhost (`"http://localhost:30658/"`). Otherwiswe this has to be an address acessible to clients.

- `config.oauthRedirectURI` this is the oauth callback address within console that the pachd oauth service would redirect to. It's the URL of console with `/oauth/callback/?inline=true` appended. Running locally its therefore `"http://localhost:4000/oauth/callback/?inline=true"`.

- `config.oauthClientID` the client identifier for the Console with pachd

- `config.oauthClientSecret` the secret configured for the client with pachd
- `config.oauthClientSecretSecretName` the secret name of the secret containing `oauthClientSecret` (key: `OAUTH_CLIENT_SECRET`)
- `config.graphqlPort` the http port that the console service will be accessible on.

- `config.disableTelemetry` this can be set to true to opt out of console's analytics and error data collection.

### etcd

This section is to configure the etcd cluster in the deployment.

- `etcd.image` sets the image to use for the etcd. This can be left at the defaults unless instructed.

- `etcd.podLabels` specifies labels to add to the etcd pods.

- `etcd.resources` specifies resources and limits in standard kubernetes format. It is left unset by default.

- `etcd.dynamicNodes` sets the number of nodes in the etcd StatefulSet. The default is 1.

- `etcd.storageClass` indicates the etcd should use an existing StorageClass for its storage. If left blank, a storageClass will be created.

- `etcd.storageSize` specifies the size of the volume to use for etcd. Etcd does not require much space. For storage that scales IOPs with size, the size must be set large enought to provide at least 1000 IOPs for performance. If you do not specify, it will default to 256Gi on Azure and 100Gi on GCP/AWS for that reason.

- `etcd.service.labels` specifies labels to add to the console service.
- `etcd.service.annotations` specifies annotations to add to the etcd service.
- `etcd.service.type` specifies the Kubernetes type of the etcd service. The default is `ClusterIP`.

### enterpriseServer

This section is to configure the Enterprise Server deployment (if desired).

- `enterpriseServer.enabled` turns on the deployment of the Enterprise Server. It is disabled by default.

- `enterpriseServer.service.type` specifies the Kubernetes type of the console service. The default is `ClusterIP`.

- `enterpriseServer.resources` specifies resources and limits in standard kubernetes format. It is left unset by default.

- `enterpriseServer.podLabels` specifies labels to add to the enterpriseServer pod.

- `enterpriseServer.image` sets the image to use for the etcd. This can be left at the defaults unless instructed.

#### enterpriseServer.tls

There are three options for configuring TLS on the Enterprise Server under `enterpriseServer.tls`.

1. `disabled`. TLS is not used.
1. `enabled`, using an existing secret. You must set enabled to true and provide a secret name where the exiting cert and key are stored.
1. `enabled`, using a new secret. You must set enabled to true and `newSecret.create` to true and specify a secret name, and a cert and key in string format

### ingress

{{% notice warning %}}
`ingress` will be removed from the helm chart once the deployment of Pachyderm with a proxy becomes mandatory.
{{% /notice %}}

This section is to configure an ingress resource for an existing ingress controller.

- `ingress.enabled` turns on the creation of the ingress for the UI.

- `ingress.annotations` specifies annotations to add to the ingress resource.

- `host` your domain name, external IP address, or localhost.

- `uriHttpsProtoOverride` when set to true, uriHttpsProtoOverride will add the https protocol to the ingress URI routes without configuring certs
  
#### ingress.tls

There are three options for configuring TLS on the ingress under `ingress.tls`.

1. `disabled`. TLS is not used.
1. `enabled`, using an existing secret. You must set enabled to true and provide a secret name where the exiting cert and key are stored.
1. `enabled`, using a new secret. You must set enabled to true and `newSecret.create` to true and specify a secret name, and a cert and key in string format


### loki-stack 

The `loki-stack` section contains all the values passed to the `loki-stack` subchart.

#### loki 

See the official [Loki storage documentation](https://grafana.com/docs/loki/latest/operations/storage/) for the most up-to-date information.

#### grafana

See the official [Grafana documentation](https://grafana.com/docs/) for the most up-to-date information.

#### promtail 

See the official [Promtail documentation](https://grafana.com/docs/loki/latest/clients/promtail/configuration/) for the most up-to-date information.



### pachd

This section is to configure the pachd deployment.

- `pachd.enabled` turns on the deployment of pachd.

- `pachd.image` sets the image to use for pachd. This can be left at the defaults unless instructed.

- `pachd.logFormat` sets the logging format (`text` or `json`). `json` is default.

- `pachd.logLevel` sets the logging level. `info` is default.

- `pachd.lokiLogging` enables Loki logging if set.

- `pachd.podLabels` specifies labels to add to the pachd pod.

- `pachd.resources` specifies resources and limits in standard kubernetes format. It is left unset by default.

- `pachd.requireCriticalServersOnly` only requires the critical pachd servers to startup and run without errors.

- `pachd.service.labels` specifies labels to add to the pachd service.

- `pachd.service.type` specifies the Kubernetes type of the pachd service. The default is `ClusterIP`.

{{% notice warning %}}
`pachd.externalService` will be removed from the helm chart once the deployment of Pachyderm with a proxy becomes mandatory.
{{%/notice%}}

- `pachd.externalService.enabled` creates a kubernetes service of type `loadBalancer` that is safe to expose externally.

- `pachd.externalService.loadBalancerIP` optionally supply the existing IP address of the load balancer.

- `pachd.externalService.apiGRPCPort` is the desired api GRPC port (30650 is default).

- `pachd.externalService.s3GatewayPort` is the desired s3 gateway port (30600 is default).

- `pachd.externalService.annotations` add your service annotations.

- `pachd.activateEnterpriseMember` specifies whether to activate with an enterprise server.
  If pachd.activateEnterpriseMember is set, enterprise will be activated and connected to an existing enterprise server.

- `activateAuth` If pachd.activateAuth is set, auth will be bootstrapped by the config-job. Defaults to true.

- `pachd.enterpriseLicenseKey` specifies the enterprise license key if you have one. 
  If pachd.enterpriseLicenseKey is set, enterprise will be activated. Alternatively, you can pass this value in a secret.

- `pachd.enterpriseLicenseKeySecretName` specifies the secret name containing pachd License Key.

- `pachd.rootToken` is the auth token used to communicate with the cluster as the root user. If a secret name is not provided in `rootTokenSecretName`, a secret containing `rootToken` (or a randomly generated value if empty) will be created on install and stored in the k8s secret `pachyderm-auth` under the key `rootToken`

- `pachd.rootTokenSecretName` specifies the secret name containing pachd root token secret (key: `rootToken`).

- `pachd.enterpriseSecret` specifies the enterprise cluster secret. If a secret name is not provided in `enterpriseSecretSecretName`, a secret containing `enterpriseSecret` (or a randomly generated value if empty) will be created on install and stored in the k8s secret `pachyderm-enterprise` under the key `enterprise-secret`

- `pachd.enterpriseSecretSecretName` specifies the secret name containing pachd enterpriseSecret (key: `enterprise-secret`).

- `pachd.oauthClientID` specifies the Oauth client ID representing pachd. Defaults to "pachd".

- `pachd.oauthClientSecret` specifies the Oauth client secret. If a secret name is not provided in `oauthClientSecretSecretName`, a secret containing `oauthClientSecret` (or a randomly generated value if empty) will be created on install and stored in the k8s secret `pachyderm-auth` under the key `auth-config`

- `pachd.oauthClientSecretSecretName` specifies the secret name containing pachd Oauth client secret (key: `auth-config`).

- `pachd.oauthRedirectURI` specifies the Oauth redirect URI served by pachd. Example  `http://<PACHD-IP>:30657/authorization-code/callback`.

- `pachd.enterpriseRootToken` only used if pachd.activateEnterpriseMember == true
- `pachd.enterpriseRootTokenSecretName` specifies the secret name containing `enterpriseRootToken`

- `pachd.enterpriseServerAddress` only used if pachd.activateEnterpriseMember == true
- `pachd.enterpriseCallbackAddress` only used if pachd.activateEnterpriseMember == true

- `pachd.localhostIssuer` specifies to pachd whether dex is embedded in its process. This value can be set to "true", "false", or "".

If any of `rootToken`,`enterpriseSecret`, or `oauthClientSecret` are blank, a value will be generated automatically. 

- `pachd.serviceAccount.create` creates a kubernetes service account for pachd. Default is true.

- `pachd.rbac.create`  indicates whether RBAC resources should be created. Default is true.

#### pachd.storage

This section of `pachd` configures the back end storage for pachyderm.

- `storage.backend` configures the storage backend to use. It must be one of GOOGLE, AMAZON, MINIO, MICROSOFT or LOCAL. This is set automatically if deployTarget is GOOGLE, AMAZON, MICROSOFT, or LOCAL.

- `storage.putFileConcurrencyLimit` sets the maximum number of files to upload or fetch from remote sources (HTTP, blob storage) using PutFile concurrently. 

- `storage.uploadConcurrencyLimit` sets the maximum number of concurrent object storage uploads per Pachd instance.
##### pachd.storage.amazon

If you're using Amazon S3 as your storage backend, configure it here.

- `storage.amazon.bucket` sets the S3 bucket to use.

- `storage.amazon.cloudFrontDistribution` sets the CloudFront distribution in the storage secrets.

- `storage.amazon.customEndpoint` sets a custom s3 endpoint.

- `storage.amazon.disableSSL` disables SSL.

- `storage.amazon.id` sets the Amazon access key ID to use.

- `storage.amazon.logOptions` sets various log options in Pachyderm’s internal S3 client.  Comma-separated list containing zero or more of: 'Debug', 'Signing', 'HTTPBody', 'RequestRetries','RequestErrors', 'EventStreamBody', or 'all' (case-insensitive).  See 'AWS SDK for Go' docs for details.

- `storage.amazon.maxUploadParts` sets the maximum number of upload parts. Default is `10000`.

- `storage.amazon.verifySSL` performs SSL certificate verification.

- `storage.amazon.partSize` sets the part size for object storage uploads. It has to be a string due to Helm and YAML parsing integers as floats.

- `storage.amazon.region` sets the AWS region to use.

- `storage.amazon.retries` sets the number of retries for object storage requests..

- `storage.amazon.reverse` reverses object storage paths.

- `storage.amazon.secret` sets the Amazon secret access key to use.

- `storage.amazon.timeout` sets the timeout for object storage requests.

- `storage.amazon.token` optionally sets the Amazon token to use.

- `storage.amazon.uploadACL` sets the upload ACL for object storage uploads.

##### pachd.storage.google

If you're using Google Storage Buckets as your storage backend, configure it here.

- `storage.google.bucket` sets the object bucket to use.

- `storage.google.cred` is a string containing a GCP service account private key, in object (JSON or YAML) form.  A simple way to pass this on the command line is with the set-file flag, e.g.:

  ```s
  helm install pachd -f my-values.yaml --set-file storage.google.cred=creds.json pach/pachyderm
  ```

  Example:

  ```yaml
  cred: |
    {
      "type": "service_account",
      "project_id": "…",
      "private_key_id": "…",
      "private_key": "-----BEGIN PRIVATE KEY-----\n…\n-----END PRIVATE KEY-----\n",
      "client_email": "…@….iam.gserviceaccount.com",
      "client_id": "…",
      "auth_uri": "https://accounts.google.com/o/oauth2/auth",
      "token_uri": "https://oauth2.googleapis.com/token",
      "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
      "client_x509_cert_url": "https://www.googleapis.com/robot/v1/metadata/x509/…%40….iam.gserviceaccount.com"
    }
  ```

- `storage.local.hostpath` indicates the path on the host where the PFS metadata will be stored.
- `storage.local.requireRoot`


##### pachd.storage.microsoft

If you're using Microsoft Blob Storage as your storage backend, configure it here.

- `storage.microsoft.container` sets the blob storage container.

- `storage.microsoft.id` sets the access key ID to use.

- `storage.microsoft.secret` sets the secret access key to use.

##### pachd.storage.minio

If you're using [MinIO](https://min.io/) as your storage backend, configure it here.

- `storage.minio.bucket` sets the bucket to use.

- `storage.minio.endpoint` sets the object endpoint.

- `storage.minio.id` sets the access key ID to use.

- `storage.minio.secret` sets the secret access key to use.

- `storage.minio.secure` set to true for a secure connection.

- `storage.minio.signature` sets the signature version to use.

#### pachd.tls

There are three options for configuring TLS on pachd under `pachd.tls`.

1. `disabled`. TLS is not used.
1. `enabled`, using an existing secret. You must set enabled to true and provide a secret name where the exiting cert and key are stored.
1. `enabled`, using a new secret. You must set enabled to true and `newSecret.create` to true and specify a secret name, and a cert and key in string format.

### pgbouncer

This section is to configure the PGBouncer Postgres connection pooler.

- `service.type` specifies the Kubernetes type of the pgbouncer service. The default is `ClusterIP`.

- `resources` specifies resources and limits in standard kubernetes format. It is left unset by default.

- `maxConnections` defaults to 1000
### postgresql

This section is to configure the PostgresQL Subchart, if used.

- `enabled`  controls whether to install postgres or not. If not using the built in Postgres, you must specify a Postgresql database server to connect to in `global.postgresql`. The enabled value is watched by the 'condition' set on the Postgresql dependency in Chart.yaml

- `image.tag` sets the postgres version. Leave at the default unless instructed otherwise.

- `initdbScripts` creates the inital `dex` database that's needed for pachyderm. Leave at the default unless instructed otherwise.

- `persistence.storageClass` specifies the storage class for the postgresql Persistent Volume (PV)

{{% notice note %}}
**More**: See notes in Bitnami chart values.yaml file for more information.
  More info for setting up storage classes on various cloud providers:

- [AWS](https://docs.aws.amazon.com/eks/latest/userguide/storage-classes.html)
- [GCP](https://cloud.google.com/kubernetes-engine/docs/how-to/persistent-volumes/ssd-pd)
- [Azure](https://docs.microsoft.com/en-us/azure/aks/concepts-storage)
{{% /notice %}}

- `storageSize` specifies the size of the volume to use for postgresql.

{{% notice warning %}}
- Recommended Minimum Disk size for Microsoft/Azure: 256Gi  - 1,100 IOPS https://azure.microsoft.com/en-us/pricing/details/managed-disks/
- Recommended Minimum Disk size for Google/GCP: 50Gi        - 1,500 IOPS https://cloud.google.com/compute/docs/disks/performance
- Recommended Minimum Disk size for Amazon/AWS: 500Gi (GP2) - 1,500 IOPS https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-volume-types.html
{{% /notice %}}

### cloudsqlAuthProxy

This section is to configure the CloudSQL Auth Proxy for deploying Pachyderm on GCP with CloudSQL.

- `connectionName` may be found by running `gcloud sql instances describe INSTANCE_NAME --project PROJECT_ID`

- `serviceAccount` is the account to use to connect to the cloudSql instance.

- `enabled`  controls whether to deploy the cloudsqlAuthProxy. Default is false.

- `port` is the cloudql database port to expose. The default is `5432`

- `service.type` specifies the Kubernetes type of the cloudsqlAuthProxy service. The default is `ClusterIP`.

### oidc

This section is to configure the oidc settings within pachyderm.

- `oidc.issuerURI` specifies the Oauth Issuer. Inferred if running locally or using ingress.

- `oidc.requireVerifiedEmail` specifies whether email verification is required for authentication.

- `oidc.IDTokenExpiry` specifies the duration where OIDC ID Tokens are valid.

- `oidc.RotationTokenExpiry` if set, enables OIDC Rotation Tokens and specifies the duration where they are valid.

- `oidc.upstreamIDPs` specifies a list of Identity Providers to use for authentication.
- `oidc.upstreamIDPsSecretName` specifies the secret name of the secret containing `upstreamIDPs` (key: `upstream-idps`)

- `oidc.mockIDP` when set to `true`, specifes to ignore `upstreamIDPs` in favor of a placeholder IDP with a username/password preset to "admin" and "password".

- `oidc.userAccessibleOauthIssuerHost` specifies the Oauth issuer's address host that's used in the Oauth authorization redirect URI. This value is only necessary in local settings or anytime the registered Issuer address isn't accessible outside the cluster.

### proxy

The proxy is a service to handle all Pachyderm traffic (S3, Console, OIDC, Dex, GRPC) on a single
port exposed directly to the Internet.

  - `proxy.enabled` when set to `true`, create a proxy deployment and a service to expose it. If
  ingress is also enabled, any ingress traffic will be routed through the proxy before being sent.
  
  - `proxy.replicas` : The number of proxy replicas to run.  1 is a default but can be increased
  for higher availability. Each replica can handle 50,000 concurrent connections. There is an affinity rule to prefer scheduling the proxy pods **on the same node as pachd**, so a number here that matches the number of pachd replicas is a fine configuration. Use this setting to scale pachd instances horizontally; horizontal scaling spreads the load from PFS across each instance.
  
{{% notice note %}}
 We don't guarantee to keep the proxy<->pachd traffic on-node or even in-region.
{{% /notice %}}

- `proxy.image` specifies the details of the proxy image to pull.  THe following fields can be left at the defaults unless instructed.

    - `proxy.image.repository` the proxy image repository. Defaults to "envoyproxy/envoy".
    - `proxy.image. tag` the tag of the proxy image.
    - `proxy.image.pullPolicy` the proxy image pull policy. Defaults to "IfNotPresent".
  
  - `proxy.resources` specifies the proxy resources. The proxy is configured to shed traffic before using 500MB of RAM, so that's a resonable memory limit.  It doesn't need much CPU.

    - `proxy.resources.requests`

      - `proxy.resources.requests.cpu` defaults to 100m
      - `proxy.resources.requests.memory` defaults to 512Mi

    - `proxy.resources.limits.memory` sets the proxy memory limit. defaults to 512Mi.

  - `proxy.labels`  list any additional labels to add to the pods.  These are also added to the deployment and service selectors.

  - `proxy.annotations` list any additional annotations to add to the pods.

  - `proxy.service` this section configure the service that routes traffic to the proxy.

     - `proxy.service.type` The type of service can be **ClusterIP**, **NodePort**, or **LoadBalancer**. Defaults to ClusterIP.

         - If the service is a **LoadBalancer**, you can specify the IP `proxy.service.loadBalancerIP` address to use, the port to serve plain HTTP traffic on `proxy.service.httpPort` (defaults to 80), the port to serve HTTPS traffic on `proxy.service.httpsPort` (defaults to 443), if enabled below.
         - If the service is a **NodePort**, you can specify the port to receive HTTP traffic on `proxy.service.httpNodePort` (defaults to 30080), and `proxy.service.httpsNodePort` (defaults to 30443).

     - `proxy.service.annotations` list any additional annotations to add.

     - `proxy.service.labels` list any additional labels to add to the service itself (not the selector!).

     - `proxy.service.legacyPorts` The proxy can also serve each backend service on a numbered port, and will do so for any port not numbered 0.  If this service is of type NodePort, the port numbers will be used for the node port, and will need to be in the node port range.
    
        - `proxy.service.legacyPorts.console` defaults to 0, legacy 30080, conflicts with default httpNodePort
        - `proxy.service.legacyPorts.grpc` defaults to 0, legacy 30650
        - `proxy.service.legacyPorts.s3Gateway` defaults to 0, legacy 30600
        - `proxy.service.legacyPorts.oidc` defaults to 0, legacy 30657
        - `proxy.service.legacyPorts.identity` defaults to 0, legacy 30658
        - `proxy.service.legacyPorts.metrics` defaults to 0, legacy 30656

  - `proxy.tls.enabled` enable TLS (SSL, HTTPS) serving when `true`.  Enabling TLS is incompatible with support for legacy ports (you can't get a generally-trusted certificate for port numbers), and disables support for cleartext communication (cleartext requests will redirect to the secure server, and HSTS headers are set to prevent downgrade attacks).

  - `proxy.tls.secretName` The secret containing "tls.key" and "tls.crt" keys that contain PEM-encoded private key and certificate material.  Generate one with "kubectl create secret tls <name> --key=tls.key --cert=tls.cert".  This format is compatible with the secrets produced by cert-manager.
 