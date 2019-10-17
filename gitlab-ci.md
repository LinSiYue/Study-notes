# GitLab-CI 可持续集成环境搭建

## 1. 安装gitlab-runner

https://docs.gitlab.com/runner/install/windows.html

之后运行：

```powershell
cd C:\GitLab-Runner
./gitlab-runner.exe install
./gitlab-runner.exe start
```

## 2.编写.gitlab-ci.yml

```shell
//前端
stages:
  - install_deps
  - build

cache:
  key: ${CI_BUILD_REF_NAME}
  paths:
    - node_modules/

job_install_deps:
  stage: install_deps
  script:
    - npm install
  tags:
    - web-tag

job_build:
  stage: build
  script:
    - npm run build
  tags:
    - web-tag // 标注之后会将tag为设置这个名字的runner分配给该任务
```

```shell
// 后端，若没有设置tags名，需要去修改runner的属性，让没有标注tags的也能分配到runner
stages:
- build
- test
- deploy

build_maven:
  stage: build
  script:
  - echo "build maven....."
  - echo "mvn clean"
  - echo "done"

test_springboot:
  stage: test
  script:
  - echo "run java test....."
  - echo "java -test"
  - echo "done"

deploy_springboot:
  stage: deploy
  script:
  - echo "deploy springboot...."
  - echo "run mvn install"
  - echo "done"
```

## 3.注册runner

![1571041612644](https://github.com/LinSiYue/Study-notes/blob/master/img/gitlab-ci/1571041612644.png?raw=true)

进入gitlab-runner.exe目录下，打开命令行窗口：

```
gitlab-runner register
```

需要输入url，然后输入token，之后输入runner的descript，再输入runner的tags名称（这个将来分配runner的时候需要用到），最后选择运行平台或方式，此处我选择shell。

