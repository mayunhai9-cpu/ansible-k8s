先解压
tar   -zxvf  ansible_k8s.tar.gz
cd ansible_k8s/playbook
ansible-playbook -i ../k8s_inventory.ini 01_init_nodes.yml
ansible-playbook -i ../k8s_inventory.ini 02_containerd.yml
ansible-playbook -i ../k8s_inventory.ini 03_keepalived_nginx.yml
ansible-playbook -i ../k8s_inventory.ini 04_k8s_helm.yml
ansible-playbook -i ../k8s_inventory.ini 05_k8s_max.yaml
ansible-playbook -i ../k8s_inventory.ini 06_k8s_cluster_join.yml
kubectl get nodes









Kubernetes v1.31.14 高可用集群自动化部署方案声明
1. 项目概述
本项目基于 Ansible 自动化工具，旨在构建一个生产级的 Kubernetes (v1.31.14) 高可用集群 。方案采用了“多 Master + 负载均衡”架构，通过 Keepalived 维护虚拟 IP (VIP)，并使用 Nginx 进行四层代理转发，确保集群控制平面的极致稳定性 。

2. 核心架构设计

高可用保障：利用 Keepalived 实现 VIP (10.11.0.88) 的故障转移，结合 Nginx 对 3 台 Master 节点的 API Server 进行负载均衡 。


运行时环境：全节点部署 Containerd 作为容器运行时，并针对 K8s 深度优化了内核模块（overlay, br_netfilter, ipvs 等）和系统参数 。

节点规划：


主 Master (10.11.0.25)：负责集群初始化、证书生成及 Helm 插件管理 。


副 Master (10.11.0.26, 27)：同步控制平面组件，实现高可用冗余 。


Worker 节点 (10.11.0.28, 29)：承载业务负载 。

3. 自动化部署流程
项目通过分层 Playbook 实现一键式部署 ：


节点初始化 (01_init_nodes.yml)：彻底锁定防火墙、清理 Swap、优化文件句柄限制（65535）并加载内核模块 。


运行时部署 (02_containerd.yml)：配置 Docker 官方源，安装并配置支持 SystemdCgroup 的 Containerd 。


高可用组件 (03_keepalived_nginx.yml)：自动化探测网卡并完成 Keepalived 与 Nginx 的四层负载均衡配置 。


核心安装 (04_k8s_helm.yml)：安装指定版本的 kubelet/kubeadm/kubectl，并为管理节点安装 Helm 工具 。


集群初始化与加入 (05_k8s_max.yaml, 06_k8s_cluster_join.yml)：主节点基于阿里云镜像源执行初始化，随后全自动分发 Token 与证书 Key，实现多节点快速加入 。

4. 环境要求

操作系统：Ubuntu (推荐 22.04/24.04) 。


连接配置：Ansible 部署机需具备目标节点的 sudo 提权权限 。


网络配置：需预留一个内网 VIP (10.11.0.88)，且各节点间 6443/16443/2379 等关键端口互通 。


声明：本脚本包已通过幂等性处理（如防火墙锁定、环境变量注入等均支持重复执行而不会产生冲突），适用于快速交付稳定的生产环境基础底座 。
