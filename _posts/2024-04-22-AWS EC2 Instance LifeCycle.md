---
title: AWS EC2 Instance LifeCycle
author: heogi
date: 2023-11-16
categories:
  - Cloud
  - EC2
tags:
  - Cloud
  - AWS
  - EC2
  - LifeCycle
comments: true
---
## **EC2 Instance LifeCycle**

![](../assets/img/Pasted%20image%2020240422144743.png)

## **Instance Status & Cost**

| **Instance Status** | **Description**                                                                        | **Instance usage Billing** |
| ------------------- | -------------------------------------------------------------------------------------- | -------------------------- |
| **Pending**         | 인스턴스가 Running 상태로 될 준비를 하고있는 상태. <br>인스턴스가 시작되거나 Stopped 상태 이후에 시작되면 Pending 상태로 전환된다. | 미청구                        |
| **Running**         | 인스턴스가 실행 중이고 사용할 준비가 된 상태                                                              | 청구                         |
| **Stopping**        | 인스턴스가 중지 상태로 전환될 준비중인 상태                                                               | 미청구                        |
| **Stopped**         | 인스턴스가 중지되고 사용할 수 없는 상태                                                                 | 미청구                        |
| **Shutting-Down**   | 인스턴스가 삭제될 준비를 하는 상태                                                                    | 미청구                        |
| **Terminated**      | 인스턴스가 영구적으로 삭제되었으며 다시 시작할 수 없는 상태                                                      | 미청구                        |
