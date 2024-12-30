Five files:
- kitt-build.yml (could be shared with others)
```
profiles:
  -  sprintboot-web-prime17-ubuntu
setup:
  featureFlagMap:
    buildWhenStageRefMatches: true
    enableIstioSidecar: true
    skipSonar: false
build:
  artifact: some_name
  buildType: maven-multimodule-j17
  attribures:
    jdkVersion: 17
    moduleVersionCommand: "-f ."
    moduleCommand: "-pl {{$.kitt.build.docker.app.buildArgs.modulePath}} -am Dsonar.projectKey={{$.kitt.build.artifact}} -Dsonar.projectNme={{$.kitt.build.artifact}}"
    mvnGoals: clean install
  docker:
    app:
      contextDir: "."
      buildArgs:
        optionalJarSuffix: "-shaded"
        zuluRuntime: 17-jdk
        zuluJDKVersion: 17-jdk-centos
        SourceDir: "./"
        modulePath: my-module-path

```
- sr.yml
```
schemaVersion: 1.0.0

notify:
  slack:
    channelName: some_slack_channel-name

applications:
  - key: CP-RST-INBOUND-OFFER-UPDATE
    name: cp-rst-inbound-offer-update
    organization: GeC
    businessCriticality: MAJOR
    environtments:
      - name: dev
        type: DEVELOPMENT
      - name: stage
        type: STAGING
      - name: prod
        type: PRODUCATION

  - key: CP-RST-INBOUND-OFFER-TAG
    name: cp-rst-inbound-offer-tag
    organization: GeC
    businessCriticality: MAJOR
    environtments:
      - name: dev
        type: DEVELOPMENT
      - name: stage
        type: STAGING
      - name: prod
        type: PRODUCATION
```

- kitt.yml
```
profiles:
  - springboot-web-prime17-ubuntu
  - git://Tunr:....ccm2v2 # it point to CCM2
  - git://...main:/kit/kitt-metrics

setup:
  releaseRefs: ["main", "hotfix/.*", "feature/.*", "release/.*"]
  featureFlagMap:
    buildWhenStageRefMatches: true
    enableIstioSidecar: true
    skipSonar: falase
owner:
  group: some_group

alerts:
  slack:
    channelName: xxx.alerts

notify:
  slack:
    channelName: wcnp-xxx

services:
  - path: xxx-kpi/kitt-build.yaml

build:
  postBuild:
    - task:
      name: deployApp
      kittFilePath: xxx-kpi/kitt-cp-kpi-service.yml
      sync: fasle

deploy:
  skip: true
  namespace: csis-xxx-async
  stages:
  - name: stage
    refEventFilters:
      - ref" ["main", "feature.*"]
        events: [onPROpen, onPRReopen, onPush, onPRSync]
      target:
        - cluster_id: [uscentral-dev-az-301]

```

- concord.yml
```
configuration:
  dependencies:
    - "mvn://org.python:jpython-standalone:2.7.2"
  arguments:
    serverApiBaseUrl: "https://concord.prod.example.com"
    gitUrl: "https://gecgithub01.example.com"
    repoOrg: "odin"
    repoName: "xxx"
    services:
      cp-kpi:
        serviceList:
          - cp-kpi/kitt-cp-kpi-service.yml
    byService:
      - cp-kpi/kitt-cp-kpi-service.yml
  approvalForm:
    - approved: {type: boolean}
  
  flows:
    default:
      - form: form1
        fields:
          - service: {label : "Service", type: "string", allow: "$service.ketSet().stream().toList()"}
          - deployAll: {label: "Deploy all sub-services?", type: "boolean"}
          - buildVersions: {label: "Enter build version", type: "string"}
      - if: ${form1.deployAll == true}
        then:
        - log: "Deploying all services for : ${form1.service}"
        - set:
          servicePath: "${services[form1.service].serviceList}"
        else:
          - form: form2
            fields:
            - toDeploy: {label: "Select one or More sub-services", type: "string+", allow: "${services[form1.service].serviceList}"}
          - set:
            servicePath : "${from2.toDpeloy}"
      - log: "Deploying servcie ; ${servicePath}"
      - set:
          children: []
          deployStage: "[prod]"
          buildVersion: ${form1.buildVersion}
          namespace: csis-cp-asyn
          prodDeploymentClusterId: [uscentral-prod-az-336]
          dev_cluster: [uscentral-dev-az-301]
          stage_cluster: [uscentral-stage-az-005]
          ...
        
```

- cp-kpi-service.yml
```
profiles:
  - git://odin:cp:main:kitt/kitt-base

build:
  artifact: cp-kpi-service

deploy:
  changeRecord:
    type: auto
    group: "USOmniTech - Derivation Platform"
    manageGroup: "Change Managers - US"
    affectedGEOs: ["US"]
    notifyChannels: ["wcnp-copernicus"]
    primaryCI: "Derivation Platform (a.k.a EXPERT)"
    affectedCIClass: "Business Application"
    risk: "4"
    impact: "4"
  namespace: csis-cp-limo
  skip: false
  gslb:
    strategy: stage
    lbRoutings:
      dev:
        cnames: ["cp-kpi-service.cp.dev.example.com"]
      stage:
        cnames: ["cp-kpi-service.cp.stage.example.com"]
      prod:
        cnames: ["cp-kpi-service.cp.prod.example.com"]
  helm:
    values:
      global:
        metrics:
          enabled: true
          remoteWriteSampleLimit: 50
          whitelist:
            - kpi
            - kpi_details
      metadata:
        labels:
          mls-cluster: mlsSct
      container:
        image: "{{$.kitt.build.docker.app.repository}}/cp/cp-kpi"
        tag: "{{$.kitt.build.version}}"
      annotation:
        sidecar.istio.io/inject: "false"
      scaling:
        enabled: false
      min:
        cpu: 1
        memory: 4G
      max:
        cpu: 1
        memory: 4G
  stages:
    - name: dev
      ...
    - name: stage
      ...
    - name: prod
      approvers:
        groups:
          - "some eng team"
      refEventFilters:
        - refs: ["main", "feature/.*"]
          events: [ onPush ]
        targets:
          - cluster_id: [uscentral-prod-az-339, uswest-prod-az-009]
        helm:
          values:
            metadata:
              mls-index: wcnp-eii-odin-mig
              labels:
                ccm.serviceId: CP-KPI-SERVICE
                ccm.serviceConfigVersion; '2.0.0'
                ccm.envProfile: 'prod'
                cc.envName: 'prod'
                wm.app: CP-KPI-SERVICE
                wm.env: prod
            secretes:
              akeyless: true
              config:
                akeyless:
                  path: /Prod/xx/xx/xx-team
                files:
                  - destination: gcp-audit.json
                    content: prod/xx/xx/gcp-audit
            scaling;
              min: 5
              max: 15
              custom: true
              enabled: true
              prometheusQueries:
                kafka-lag:
                  queryContent: max by (topic,consumerGroup, cloudRegion) (kafka_total_lag{job="", cluster_profile={{$.kittExec.currentCluster.profile}}}) * on() 
                  max(kube_deployment_replicas{job="", cluster_profile={{$.kittExec.currentCluster.profile}} ...}) by (cloud_region)
                  targetAverageValue: 100000
              customBehaviorEnabled: true
              behavior:
                scaleDown:
                  stabilizationWindowSeconds: 3600
                  policies:
                    - type: Pods
                      value: 1
                      periodSeconds: 60
                scaleUp:
                  stabilizationWindowSeconds: 180
                  policies:
                    - type: Pods
                      value: 2
                      periodSeconds: 90
                  selectPolicy: Max
                min:
                  cpu: 4
                  memory: 8G
                max:
                  cpu: 4
                  memory: 8G

            env:
              GOOGLE_APPLICATION_CREDENTIALS: /etc/secret/gcp-audit.json
              JAVA_OPTS: >-
                -Xms6000m
                -Xmx6000m
                -XX:+UseG1GC
                -XX:+UseStringDeduplication
                -Xlog:gc**"file=/tmp/gc.log:utctime,pid,level,tags,uptime:filecount=5,filesize=50M
                -Dccm.configs.dir=/etc/config
                -Dmicronaut.environments=k8s
                -Dmicronaut.config.files=/etc/config/appConfig.properties
                -Dcom.sun.managemnt.jmxremote.rmi.port=9090
                -Dcom.sun.managemnt.jmxremote=true
                -Dcom.sun.managemnt.jmxremote.port=9090
                -Dcom.sun.managemnt.jmxremote.ssl=true
                -Dcom.sun.managemnt.jmxremote.authenticate=false
                -Dcom.sun.managemnt.jmxremote.local.only=fasle
                -Djava.rmi.server.hostname=localhost
    
```

- kitt-metrics.yml
```
deploy:
  helm:
    values:
      global:
        metrics:
          enabled: true
          endpoints:
            - targetPort: 8080
              path: "prometheus"
          remoteWriteSampleLimit: 100
          whitelist:
            - kpi
            - kpi_details
```

- ccm
```
---
metadata:
  serviceId: "CP-KPI-SERVICE"
  serviceConfigVersion: "2.0.0"
  tags:
    - "cp-kpi-service"
  authzPolicies:
    adminUI:
      adminGroups:
        - "eng_team"
      git:
        org: "odin"
        repo: "cp-configs"
        brach: "main"
  externallyReferencedServiceConfig:
    - serviceId: "ccm-app"
      serviceConfigVersions: "NON-PROD-1.0"
  notification:
    slack:
      channel: "cp-alerts"
  deliveryEnvironment: "NON-PROD"
  locked: false

configDefinition:
  appConfig:
    deliveryType: "PROPERTIES"
    description:  "App Configuration"
    resolutionPaths:
      - default: "/envProfile/envName"
    properties:
      cp.error.retryCount:
        description: ""
        type: "INTEGER"
        kind: "SINGLE"
        defaultValue: 10000
      cp.error.retryDelay:
        description: ""
        type: "STRING"
        kind: "SINGLE"
        defaultValue: 30s
```