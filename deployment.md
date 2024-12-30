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

```