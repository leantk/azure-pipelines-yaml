# Triggers in pipelines

Triggers are events on which you can start your pipeline run automatically. You can enable triggers on your pipeline by subscribing to both internal and external events.

At high level there are 3 different types of pipeline triggers.
1. Resource triggers
2. Schedule triggers
3. Webhook triggers

## Resource triggers
You can enable triggers on the resources defined in your pipeline. Resources can be of types pipelines, repositories, containers and packages.
Triggers are enabled by default on all the resources. However, you can choose to override/disable triggers for each resource.



### Pipelines
A new pipeline is triggered automatically whenever a new run of the `pipeline` resource is succesfully completed. 

#### Scenarios
- I would like to trigger my pipeline when a new ‘helm-package’ artifact is published by ‘Helm-CI’ pipeline that ran on `releases/*` branch.
- I would like to trigger my pipeline when a new ‘helm-package’ artifact is published and tested as part of Helm-CI pipeline and tagged as 'Production'.
- I would like to trigger my pipeline when ‘TFS-Update’ pipeline has completed ‘Ring2’ stage.  

#### Schema
```yaml
resources:       
  pipelines:
  - pipeline: string 
    source: string  
    trigger:     # Optional; Triggers are enabled by default.
      branches:  # branch conditions to filter the events, optional; Defaults to all branches.
        include: [ string ]  # branches to consider the trigger events, optional; Defaults to all branches.
        exclude: [ string ]  # branches to discard the trigger events, optional; Defaults to none.
      stages: [ string ]  # trigger after completion of given stage, optional; Defaults to all stage completion.
      tags: [ string ]  # tags on which the trigger events are considered, optional; Defaults to any tag or no tag.
```

#### Examples
Triggers are enabled by default unless expliciltly opted out. You can disable the triggers on the `pipeline` resource.
```yaml
resources:
  pipelines:
  - pipeline: SmartHotel
    source: SmartHotel-CI 
    trigger: none   
```


You can control which branches get the triggers with a simple syntax.
```yaml
resources:
  pipelines:
  - pipeline: SmartHotel
    source: SmartHotel-CI 
    trigger: 
    - releases/*
    - master
 ```
 You can specify the full name of the branch (for example, master) or a prefix-matching wildcard (for example, releases/*). You cannot put a wildcard in the middle of a value. For example, releases/*2018 is invalid.
 


You can specify the branches to include and exclude.
```yaml
resources:
  pipelines:
  - pipeline: SmartHotel
    source: SmartHotel-CI 
    trigger: 
      branches:
        include: 
        - releases/*
        exclude:
        - master
 ```
 
 
 You can specify which tags to control the triggers.
 ```yaml
resources:
  pipelines:
  - pipeline: SmartHotel
    source: SmartHotel-CI 
    trigger: 
      branches:
        include: 
        - releases/*
      tags: 
      - Production
      - Signed
 ```
 
 
If you don't want to wait until all the stages of the run are completed for the `pipeline` resource. You can provide the stage to be completed to trigger you pipeline. 
```yaml
resources:
  pipelines:
  - pipeline: SmartHotel
    source: SmartHotel-CI 
    trigger: 
      branches:
        include: master
      stages: 
      - QA
 ```
 
 
### Repositories
Triggers can be set on `repository` resources defined the pipeline. For repositories, you can set two types of triggers.
1. Repo triggers
2. PR triggers


#### Repo triggers
Whenever a commit goes to your `repository`, a new pipeline run gets triggered. 


#### Scenarios:	
- I would like to trigger my pipeline only when a commit happens on ‘releases/*’ branch of the repository.
- I would like to trigger my pipeline when a new commit happens, however, I would like to enable batching so that only one pipeline runs at a time. 
- I would like to trigger my pipeline only when a new commit goes into the file path “Repository/Web/*”.


#### Schema
```yaml
resources:       
  repositories:
  - repository: string    
    type: enum 
    connection: string 
    source: string  
    trigger:  # Optional; Triggers are enabled by default
      batch: boolean 
      branches:  # branch conditions to filter the events, optional; Defaults to all branches.
        include: [ string ]  # branches to consider the trigger events, optional; Defaults to all branches.
        exclude: [ string ]  # branches to discard the trigger events, optional; Defaults to none.  
      paths:
        include: [ string ]  # file paths to consider the trigger events, optional; Defaults to all branches.
        exclude: [ string ]  # file paths to discard the trigger events, optional; Defaults to none.  
```

#### Examples : Repo triggers

Triggers are enabled by default and any new change to your repo will trigger a new pipeline run automatically. You can disable triggers on your repository.

```yaml
resources:         
  repositories:
  - repository: secondaryRepo      
    type: GitHub
    connection: myGitHubConnection
    source: Microsoft/alphaworz
    trigger: none
```

You can control which branches to get triggers with simple syntax.
```yaml
resources:         
  repositories:
  - repository: myPHPApp      
    type: GitHub
    connection: myGitHubConnection
    source: ashokirla/phpApp
    trigger:
      - master
      - releases/*
```

You can specify branches and paths to include and exclude.
```yaml
resources:         
  repositories:
  - repository: myPHPApp      
    type: GitHub
    connection: myGitHubConnection
    source: ashokirla/phpApp
    trigger:
      branches:
        include:
        - features/*
        exclude:
        - features/experimental/*
      paths:
        exclude:
        - README.md
```


If you have a lot of team members uploading changes often, then you might want to reduce the number of builds you're running. If you set batch to true, when a build is running, the system waits until the build is completed, then queues another build of all changes that have not yet been built.

```yaml
resources:         
  repositories:
  - repository: myPHPApp      
    type: GitHub
    connection: myGitHubConnection
    source: ashokirla/phpApp
    trigger:
      batch: true
      branches:
        - master
```



#### PR triggers
Whenever a PR is raised on the repository, you can choose to trigger your pipeline using PR triggers. PR triggers are not enabled by defaut. You can enable PR triggers on the repository by defining `pr` trigger on the `repository` resource.


#### Scenarios
	I would like to trigger my pipeline only when a PR is targeted to `releases/*` branch of the repository.
	I would like to trigger my pipeline only when a PR is targeted to the file path `Web/*` of the repository.


#### Schema
```yaml
resources:         
  repositories:
  - repository: string    
    source: string  
    pr:        # Optional; pr triggers are disabled by default
      branches:  # branch conditions to filter the events, optional; Defaults to all branches.
        include: [ string ]  # branches to consider the trigger events, optional; Defaults to all branches.
        exclude: [ string ]  # branches to discard the trigger events, optional; Defaults to none.  
      paths:
        include: [ string ]  # file paths to consider the trigger events, optional; Defaults to all branches.
        exclude: [ string ]  # file paths to discard the trigger events, optional; Defaults to none.
```

#### Examples: PR triggers
Unless you specify, `pr` triggers are disabled for your repository. You can enable pull request based pipeline runs.

```yaml
resources:         
  repositories:
  - repository: myPHPApp      
    type: GitHub
    connection: myGitHubConnection
    source: ashokirla/phpApp
    pr: true
```

You can control the target branches for your pull request based pipeline runs by simple syntax.
```yaml
resources:         
  repositories:
  - repository: myPHPApp      
    type: GitHub
    connection: myGitHubConnection
    source: ashokirla/phpApp
    pr: 
    - master
    - releases/*
```

You can specify the branches and file paths to include and exclude.
```yaml
resources:         
  repositories:
  - repository: myPHPApp      
    type: GitHub
    connection: myGitHubConnection
    source: ashokirla/phpApp
    pr:
      branches:
        include:
        - releases/*
        exclude:
        - releases/experimental/*
      paths:
        include:
        - web/*
        exclude:
        - web/README.md
```



### Containers
Whenever a new image got published to the container registry, your pipeline run will be triggered. 


#### Scenarios	
- I would like to trigger my pipeline whenever a new image got published.
- I would like to trigger my pipeline whenever a new image got published to ‘East-US’ location (ACR specific filter).


#### Schema
```yaml
resources:          
  containers:
  - container: string       
    type: enum  
    connection: string 
    image: string # container image name, Tag/Digest is optional; defaults to latest image
    trigger: # Optional; Triggers are enabled by default
      tags:
        include: [ string ]  # image tags to consider the trigger events, optional; defaults to any new tag
        exclude: [ string ]  # image tags on discard the trigger events, option; defaults to none
      location: [ string ]  # geo-location the image published to; ACR specific setting.
```

#### Examples

Triggers are enabled by default on the `container` resource. When a new image gets published to your image registry, pipeline run starts automatically. You can disable the triggers on your `container`.

```yaml
resources:         
  containers:
  - container: smartHotel 
    type: Docker
    connection: myDockerRegistry
    image: smartHotelApp 
    trigger: none
```


You can specify the image tag format to get the trigger by simple syntax.
```yaml
resources:         
  containers:
  - container: smartHotel 
    type: Docker
    connection: myDockerRegistry
    image: smartHotelApp 
    trigger:
    - version-*
```


You can specify the image tags to include and exclude.
```yaml
resources:         
  containers:
  - container: smartHotel 
    type: Docker
    connection: myDockerRegistry
    image: smartHotelApp 
    trigger:
      tags:
        include:
        - version-*
        exclude:
        - version-2017*
```


If you have an ACR `container` resource, you can specify the geo location to get the triggers.
```yaml
repositories:
  containers:
  - container: MyACR    
    connection: RMPM
    registry: contosodemo
    image: Microsoft/alphaworz
    trigger: 
      tags:
        include: 
        - production*
      location: 
      - east-US
      - west-EU
```