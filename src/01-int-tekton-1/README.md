# Tekton: Up and Running


## Puspose

  - Have a look tekton
  - Install tekton to our dev cluster
  - Install task and pipeline


## What is tekton

Tekton is a powerful yet flexible Kubernetes-native open source framework for creating continuous integration and delivery (CI/CD) systems. It lets you build, test, and deploy across multiple cloud providers or on-premises systems. [More info...](https://tekton.dev/)


## Tekton Pipelines entities

<table>
  <tr>
    <th>Entity</th>
    <th>Description</th>
  </tr>
  <tr>
    <td><code>Task</code></td>
    <td>Defines a series of steps which launch specific build or delivery tools that ingest specific inputs and produce specific outputs.</td>
  </tr>
  <tr>
    <td><code>TaskRun</code></td>
    <td>Instantiates a <code>Task</code> for execution with specific inputs, outputs, and execution parameters. Can be invoked on its own or as part of a <code>Pipeline</code>.</td>
  </tr>
  <tr>
    <td><code>Pipeline</code></td>
    <td>Defines a series of <code>Tasks</code> that accomplish a specific build or delivery goal. Can be triggered by an event or invoked from a <code>PipelineRun</code>.</td>
  </tr>
  <tr>
    <td><code>PipelineRun</code></td>
    <td>Instantiates a <code>Pipeline</code> for execution with specific inputs, outputs, and execution parameters.</td>
  </tr>
  <tr>
    <td><code>PipelineResource (Deprecated)</code></td>
    <td>Defines locations for inputs ingested and outputs produced by the steps in <code>Tasks</code>.</td>
  </tr>
  <tr>
    <td><Code>Run</code> (alpha)</td>
    <td>Instantiates a Custom Task for execution when specific inputs.</td>
  </tr>
</table>


## Install tekton-operator and pipelines

Requirements: 
  - You must have a Kubernetes cluster running version 1.22 or later.
  - then Choose the version
    - Official - install this unless you have a specific reason to go for a different release
    - Nightly - may contain bugs, install at your own risk.

```
kubectl apply --filename https://storage.googleapis.com/tekton-releases/pipeline/latest/release.yaml
```

You can monitor your install with:
```
kubectl apply -f https://storage.googleapis.com/tekton-releases/operator/latest/release.yaml
```

```
kubectl apply -f https://storage.googleapis.com/tekton-releases/pipeline/latest/release.yaml
```


## Install tkn (CLI)

[Release Page](https://github.com/tektoncd/cli/releases)

Debian, Ubuntu, and other deb-based distros
```
curl -LO LINK-TO-THE-PACKAGE
sudo dpkg -i ./PACKAGE-NAME
```

Fedora, CentOS, and other rpm-based distros
```
# Replace LINK-TO-THE-PACKAGE with the package URL you would like to use.
rpm -Uvh LINK-TO-THE-PACKAGE
```

If you are using the latest releases of Fedora or RHEL or CentOS, you may use the TektonCD CLI unofficial copr package repository instead:
```
dnf copr enable chmouel/tektoncd-cli
dnf install tektoncd-cli
```

Alternatively, you may download tkn as a tarball:
```
# Replace LINK-TO-TARBALL with the package URL you would like to use.
curl -LO LINK-TO-TARBALL
# Replace YOUR-DOWNLOADED-FILE with the file path of your own.
sudo tar xvzf YOUR-DOWNLOADED-FILE -C /usr/local/bin/ tkn
```

then run tkn:
```
CLI for tekton pipelines

Usage:
tkn [flags]
tkn [command]


Available Commands:
  bundle*               Manage Tekton Bundles (experimental)
  chain                 Manage Chains
  clustertask           Manage ClusterTasks
  clustertriggerbinding Manage ClusterTriggerBindings
  eventlistener         Manage EventListeners
  hub                   Interact with tekton hub
  pipeline              Manage pipelines
  pipelinerun           Manage PipelineRuns
  resource              Manage pipeline resources
  task                  Manage Tasks
  taskrun               Manage TaskRuns
  triggerbinding        Manage TriggerBindings
  triggertemplate       Manage TriggerTemplates

Other Commands:
  completion            Prints shell completion scripts
  version               Prints version information

Flags:
  -h, --help   help for tkn

Use "tkn [command] --help" for more information about a command.
```
or
```
kube@DEVK8S01:~$ tkn task list
No Tasks found
```


## Create our first Task

hello-world.yaml
```
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: hello-world
spec:
  steps:
    - name: our-step
      image: registry.hub.docker.com/library/busybox
      command:
        - sh
      args: ['-c', 'echo Hello World']
```


## Create our first Pipeline

task02.yaml
```
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: tell-the-world
spec:
  params:
    - name: message
      description: The most important message ever
      default: hello-world
      type: string
    - name: pause-duration
      description: How long to wait before saying something
      default: 0
      type: string
  steps:
    - name: first-step
      image: registry.hub.docker.com/library/busybox
      command:
        - sh
      args: ['-c', 'sleep $(params.pause-duration) && echo $(params.message)']
```

pipeline02.yaml
```
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: tell-the-world
spec:
  tasks:
    - name: task-01
      params:
        - name: pause-duration
          value: "1"
        - name: message
          value: "Lets start our first task"
      taskRef:
        name: tell-the-world
    - name: task-02
      params:
        - name: message
          value: "Finish our pipeline with our second task"
      taskRef:
        name: tell-the-world
```

pipeline03.yaml
```
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: tell-the-world-parallel
spec:
  tasks:
    - name: task-01
      params:
        - name: pause-duration
          value: "1"
        - name: message
          value: "Lets start our first task"
      taskRef:
        name: tell-the-world
    - name: task-02
      params:
        - name: message
          value: "Run our pipeline with our second task"
      taskRef:
        name: tell-the-world
      runAfter:
        - task-01
    - name: task-03
      params:
        - name: message
          value: "Finish our pipeline with our 3rd task"
      taskRef:
        name: tell-the-world
      runAfter:
        - task-02
    - name: task-04
      params:
        - name: message
          value: "Finish our pipeline with our 4th task"
      taskRef:
        name: tell-the-world
      runAfter:
        - task-02
```


## Install Tekton Dashboard

To install Tekton Dashboard on a Kubernetes cluster:
```
kubectl apply --filename https://storage.googleapis.com/tekton-releases/dashboard/latest/release.yaml
```
