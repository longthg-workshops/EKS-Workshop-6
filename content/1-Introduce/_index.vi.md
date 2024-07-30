---
title: "Giới thiệu"
date: "`r Sys.Date()`"
weight: 1
chapter: false
pre: "<b> 1. </b>"
---

#### Giới thiệu

#### Hiểu về mạng trong Kubernetes

Hiểu về mạng trong Kubernetes là vô cùng quan trọng để vận hành cụm và ứng dụng của bạn một cách hiệu quả. Trong chương này, chúng ta sẽ đào sâu vào các khía cạnh khác nhau của mạng trong Kubernetes, bao gồm mạng Pod, mạng dịch vụ, và giao tiếp dịch vụ.

Trong Amazon EKS, mạng Pod, còn được gọi là mạng cụm, được giải quyết thông qua việc sử dụng một plugin CNI của Kubernetes gọi là Amazon VPC CNI. Chúng tôi rất khuyến khích khám phá các tùy chọn khác nhau có sẵn với Amazon VPC CNI trước khi chuyển sang Amazon VPC Lattice.

- Các phần cần tìm hiểu 
    - [Switching, Routing and Gateways]( {{< relref "./1.1-Switching Routing Gateways/_index.vi.md" >}})
      - [Switching]({{< relref "./1.1-Switching Routing Gateways/_index.vi.md/#chuyển-đổi-switching" >}})
      - [Routing]({{< relref "./1.1-Switching Routing Gateways/_index.vi.md/#định-tuyến-routing" >}})
      - [Default Gateway]({{< relref "./1.1-Switching Routing Gateways/_index.vi.md/#cổng-kết-nối-gateways" >}})
    - [DNS]( {{< relref "./1.2-DNS/_index.vi.md" >}})
      - [DNS Configuration on Linux]( {{< relref "./1.2-DNS/_index.vi.md/#dns" >}})
    - [CoreDNS]( {{< relref "./1.3-CoreDNS/_index.vi.md" >}})
    - [Network Namespaces]( {{< relref "./1.4-Network Namespaces/_index.vi.md" >}})
    - [Xây dựng mạng Docker]( {{< relref "./1.5-Docker Networking/_index.vi.md" >}})
    - [CNI]( {{< relref "./1.6-CNI/_index.vi.md" >}})
    - [Xây dựng mạng Cluster]( {{< relref "./1.7-Cluster Networking/_index.vi.md" >}})
    - [Xây dựng mạng Pod]( {{< relref "./1.8-Pod Networking/_index.vi.md" >}})
    - [CNI trong Kubernetes]( {{< relref "./1.9-CNI in Kubernetes/_index.vi.md" >}})
    - [CNI weave]( {{< relref "./1.10-CNI weave/_index.vi.md" >}})
    - [IPAM weave]( {{< relref "./1.11-IPAM weave/_index.vi.md" >}})
    - [Xây dựng mạng dịch vụ]( {{< relref "./1.9-CNI in Kubernetes/_index.vi.md" >}})
    - [DNS trong Kubernetes]( {{< relref "./1.12-Service Networking/_index.vi.md" >}})
    - [CoreDNS trong Kubernetes]( {{< relref "./1.13-DNS in Kubernetes/_index.vi.md" >}})
    - [Ingress]( {{< relref "./1.15-Ingress/_index.vi.md" >}})
  