* 修改docker cgroup driver为systemd
```bash
$ sudo bash -c "cat > /etc/docker/daemon.json" <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"]
}
EOF
$ sudo systemctl restart docker
```
* 初始化主节点 the number of available CPUs 1 is less than the required 2
> 初始化添加参数 --ignore-preflight-errors=NumCPU