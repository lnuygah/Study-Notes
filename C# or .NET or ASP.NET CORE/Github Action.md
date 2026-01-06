# Github Actions

## 1. 什么是Github Actions

GItHub Actions是一个[持续集成](https://so.csdn.net/so/search?q=持续集成&spm=1001.2101.3001.7020)和持续交付的平台，能够让你自动化你的编译、测试和部署流程。

GitHub 提供 Linux、Windows 和 [macOS](https://so.csdn.net/so/search?q=macOS&spm=1001.2101.3001.7020) 虚拟机来运行您的工作流程，或者您**可以在自己的数据中心或云基础架构**中托管自己的自托管运行器。

## 2. Github Actions 的组成？

整体流程：

在github repo中特定事件发生时，workflow会被触发。

一个workflow由若干个job组成，这些job可以顺序运行，也可以并行运行。

每个job运行在自己的虚拟机runner或者容器中。

每个job由若干个step组成，这些step可以是运行脚本或者 执行一个action。

action是可以复用的拓展，用来简化workflow 。



`Workflows`

Workflow是可配置的自动化进程，它会运行一个或多个Jobs。
Workflow通过yaml配置文件来定义
Worklofw的触发方式有两种：
1.手动触发 2. 通过定义触发事件自动触发 3. 通过Post 一个Rest API请求。
一个repo可以有多个Workflow
可以在一个Workflow中引用另一个Workflow

`Events`

Events是一种能够触发Workflow运行的特定的活动，包括创建pull请求，开启一个issue或者进行一次commit，主要是体现于yml文件上面的on关键字，通过此定义触发workflows的Events。

`Jobs`

Job是Step的集合，一个Job包含若干个Steps。
每个Steps可以是将被执行的Shell命令或脚本。
不同的Jobs可以在不同的虚拟环境runner上执行若干个命令或脚本。
但一个Job只能在相同的runner上执行workflow的一系列步骤，正因如此，一个Job中的若干个Steps可以共享数据。
Job之间可以配置依赖，默认情况下Jobs彼此之间没有依赖且并行执行。
在配置了依赖的情况下，一个Job需要等待它所依赖的Job运行完成后才能运行。

`Acitons`

Action是Github Acitons 平台自定义的应用，可以执行复杂但频繁重复的任务。
也就是说将重复使用的Workflow抽象成一个模块，方便重复使用，减少重复性代码。

`Runners`

Runner是在workflow被触发时用来运行workflow的服务器。
每个Runner每次只能运行一个Job
Github提供了Ubuntu Linux，Microsoft Windows， macO runner来运行workflows。
每个workflow在全新的初始化的虚拟机上执行。

如果有需要，还可以使用自己的服务器主机来运行workflow。



`Example`**：helloWorkflow.yaml**

```yaml
name: learn-github-actions
on: [push]
jobs:
  check-bats-version:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v2
        with:
          node-version: '14'
      - run: npm install -g bats
      - run: bats -v

```

name : Workflow的名字

on: 触发的Events，可以是一个Events数组

jobs ：指定这个Workflow包含的Jobs

Check-bats-version：job的名字

runs-on：指定运行的Runner

steps：Job中包含的Steps

uses：使用某个action

run：执行某个shell命令或脚本

with：指定需要传入的参数
