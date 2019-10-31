# Installation of vSphere on OCP4
# Pre-Reqs
This repository follows the following article.
[https://blog.openshift.com/deploying-a-user-provisioned-infrastructure-environment-for-openshift-4-1-on-vsphere/](https://blog.openshift.com/deploying-a-user-provisioned-infrastructure-environment-for-openshift-4-1-on-vsphere/)

## Pre-Reqs
### HAPROXY ( instead of using DNS)
yum install -y haproxy
cat > /etc/haproxy/haproxy.cfg <<EOF
#---------------------------------------------------------------------
# Global settings
#---------------------------------------------------------------------
global
    # to have these messages end up in /var/log/haproxy.log you will
    # need to:
    #
    # 1) configure syslog to accept network log events.  This is done
    #    by adding the '-r' option to the SYSLOGD_OPTIONS in
    #    /etc/sysconfig/syslog
    #
    # 2) configure local2 events to go to the /var/log/haproxy.log
    #   file. A line like the following can be added to
    #   /etc/sysconfig/syslog
    #
    #    local2.*                       /var/log/haproxy.log
    #
    log         127.0.0.1 local2

    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     4000
    user        haproxy
    group       haproxy
    daemon

    # turn on stats unix socket
    stats socket /var/lib/haproxy/stats

#---------------------------------------------------------------------
# common defaults that all the 'listen' and 'backend' sections will
# use if not designated in their block
#---------------------------------------------------------------------
defaults
    mode                    http
    log                     global
    option                  httplog
    option                  dontlognull
    option http-server-close
    option forwardfor       except 127.0.0.0/8
    option                  redispatch
    retries                 3
    timeout http-request    10s
    timeout queue           1m
    timeout connect         10s
    timeout client          1m
    timeout server          1m
    timeout http-keep-alive 10s
    timeout check           10s
    maxconn                 3000

#---------------------------------------------------------------------

listen stats
    bind :9000
    mode http
    stats enable
    stats uri /
    monitor-uri /healthz


frontend openshift-api-server
    bind *:6443
    default_backend openshift-api-server
    mode tcp
    option tcplog

backend openshift-api-server
    balance source
    mode tcp
    server bootstrap ${BOOTSTRAP_IP}:6443 check
    server master0 ${MASTER_IP}:6443 check
    server master1 ${MASTER_IP}:6443 check
    server master2 ${MASTER_IP}:6443 check

frontend machine-config-server
    bind *:22623
    default_backend machine-config-server
    mode tcp
    option tcplog

backend machine-config-server
    balance source
    mode tcp
    server bootstrap ${BOOTSTRAP_IP}:22623 check
    server master0 ${MASTER_IP}:22623 check
    server master1 ${MASTER_IP}2:22623 check
    server master2 ${MASTER_IP}:22623 check

frontend ingress-http
    bind *:80
    default_backend ingress-http
    mode tcp
    option tcplog

backend ingress-http
    balance source
    mode tcp
    server worker0-http-router0 ${COM80 check
    server worker1-http-router1 192.168.1.55:80 check
    server worker2-http-router2 192.168.1.56:80 check

frontend ingress-https
    bind *:443
    default_backend ingress-https
    mode tcp
    option tcplog

backend ingress-https
    balance source
    mode tcp
    server worker0-https-router0 192.168.1.54:443 check
    server worker1-https-router1 192.168.1.55:443 check
    server worker2-https-router2 192.168.1.56:443 check

#---------------------------------------------------------------------
EOF

### DNS ( Can LoadBalance for testing )
    yum install -y named
    cat > /var/named/zonefile.db << EOF
    $ORIGIN apps.upi.example.com. 
    * A 10.x.y.38 
    * A 10.x.y.39 
    * A 10.x.y.40 
    $ORIGIN upi.example.com. 
    _etcd-server-ssl._tcp SRV 0 10 2380 etcd-0 
    _etcd-server-ssl._tcp SRV 0 10 2380 etcd-1 
    _etcd-server-ssl._tcp SRV 0 10 2380 etcd-2 
    bootstrap-0 A 10.x.y.34 
    control-plane-0 A 10.x.y.35 
    control-plane-1 A 10.x.y.36 
    control-plane-2 A 10.x.y.37 
    api A 10.x.y.34 
    api A 10.x.y.35 
    api A 10.x.y.36 
    api A 10.x.y.37 
    api-int A 10.x.y.34
    api-int A 10.x.y.35
    api-int A 10.x.y.36 
    api-int A 10.x.y.37 
    etcd-0 A 10.x.y.35 
    etcd-1 A 10.x.y.36 
    etcd-2 A 10.x.y.37 
    compute-0 A 10.x.y.38 
    compute-1 A 10.x.y.39 
    compute-2 A 10.x.y.40
    EOF

## Installation
git clone git@github.com:hornjason/ocp4-vsphere.git
cd ocp4-vsphere/
cp terraform.tfvars.example terraform.tfvars
cp ocp-4.2/install-config.yaml.example  ocp-4.2/install-config.yaml; cp ocp-4.2/install-config.yaml .

// Install terraform 11.0.2 and jq
// jq is in epel 
yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
yum install -y jq
// Terraform 11.02 , repo doesn't work with newer versions of terraform
wget https://releases.hashicorp.com/terraform/0.11.0/terraform_0.11.0_linux_amd64.zip && unzip terraform_0.11.0_linux_amd64.zip -d /usr/local/bin
// Edit install-config.yaml to generate ignition files

// Edit terraform.tfvars

// Run installation




git clone [https://github.com/hornjason/ocp4-vsphere](https://github.com/hornjason/ocp4-vsphere)

cd ocp4-vsphere
edit machine/ignition.tf;
change gw = "local gw"
change DNS1 = "dns server"


    // ID identifying the cluster to create. Use your username so that resources created can be tracked back to you.
    cluster_id = "example-cluster"
    
    // Domain of the cluster. This should be "${cluster_id}.${base_domain}".
    cluster_domain = "example-cluster.devcluster.openshift.com"
    
    // Base domain from which the cluster domain is a subdomain.
    base_domain = "devcluster.openshift.com"
    
    // Name of the vSphere server. The dev cluster is on "vcsa.vmware.devcluster.openshift.com".
    vsphere_server = "vcsa.vmware.devcluster.openshift.com"
    
    // User on the vSphere server.
    vsphere_user = "YOUR_USER@SSO.DOMAIN"
    
    // Password of the user on the vSphere server.
    vsphere_password = "YOUR_PASSWORD"
    
    // Name of the vSphere cluster. The dev cluster is "devel".
    vsphere_cluster = "devel"
    
    // Name of the vSphere data center. The dev cluster is "dc1".
    vsphere_datacenter = "dc1"
    
    // Name of the vSphere data store to use for the VMs. The dev cluster uses "nvme-ds1".
    vsphere_datastore = "nvme-ds1"
    
    // Name of the vSphere folder that will be created and used to store all created objects.
    vsphere_folder = "Folder Name"
    
    // Name of the VM template to clone to create VMs for the cluster. The dev cluster has a template named "rhcos-latest".
    vm_template = "rhcos-latest"
    
    // The machine_cidr where IP addresses will be assigned for cluster nodes.
    // Additionally, IPAM will assign IPs based on the network ID.  This is the network where VMs will live
    machine_cidr = "10.0.0.0/24"
    
    // The number of control plane VMs to create. Default is 3.
    control_plane_count = 3
    
    // The number of compute VMs to create. Default is 3.
    compute_count = 3
    
    // URL of the bootstrap ignition. This needs to be publicly accessible so that the bootstrap machine can pull the ignition.
    bootstrap_ignition_url = "URL_FOR_YOUR_BOOTSTRAP_IGNITION"
    
    // These are created by creating the install-config.yaml and running openshift-install create ignition-configs
    // Ignition config for the control plane machines. You should copy the contents of the master.ign generated by the installer.
    control_plane_ignition = <<END_OF_MASTER_IGNITION
    Copy the master ignition generated by the installer here.
    END_OF_MASTER_IGNITION
    
    // Ignition config for the compute machines. You should copy the contents of the worker.ign generated by the installer.
    compute_ignition = <<END_OF_WORKER_IGNITION
    Copy the worker ignition generated by the installer here.
    END_OF_WORKER_IGNITION
    
    // Set ipam and ipam_token if you want to use the IPAM server to reserve IP
    // addresses for the VMs.
    
    // Address or hostname of the IPAM server from which to reserve IP addresses for the cluster machines.
    //ipam = "139.178.89.254"
    
    // Token to use to authenticate with the IPAM server.
    //ipam_token = "TOKEN_FOR_THE_IPAM_SERVER"
    
    
    // Set bootstrap_ip, control_plane_ip, and compute_ip if you want to use static
    // IPs reserved someone else, rather than the IPAM server.
    
    // The IP address to assign to the bootstrap VM.
    // These are on the same network as 'machine_cidr'
    bootstrap_ip = "10.0.0.10"
    
    // The IP addresses to assign to the control plane VMs. The length of this list
    // must match the value of control_plane_count.
    control_plane_ips = ["10.0.0.20", "10.0.0.21", "10.0.0.22"]
    
    // The IP addresses to assign to the compute VMs. The length of this list must
    // match the value of compute_count.
    compute_ips = ["10.0.0.30", "10.0.0.31", "10.0.0.32"]

\
## SmartyPants

SmartyPants converts ASCII punctuation characters into "smart" typographic punctuation HTML entities. For example:

|                |ASCII                          |HTML                         |
|----------------|-------------------------------|-----------------------------|
|Single backticks|`'Isn't this fun?'`            |'Isn't this fun?'            |
|Quotes          |`"Isn't this fun?"`            |"Isn't this fun?"            |
|Dashes          |`-- is en-dash, --- is em-dash`|-- is en-dash, --- is em-dash|


## KaTeX

You can render LaTeX mathematical expressions using [KaTeX](https://khan.github.io/KaTeX/):

The *Gamma function* satisfying $\Gamma(n) = (n-1)!\quad\forall n\in\mathbb N$ is via the Euler integral

$$
\Gamma(z) = \int_0^\infty t^{z-1}e^{-t}dt\,.
$$

> You can find more information about **LaTeX** mathematical expressions [here](http://meta.math.stackexchange.com/questions/5020/mathjax-basic-tutorial-and-quick-reference).


## UML diagrams

You can render UML diagrams using [Mermaid](https://mermaidjs.github.io/). For example, this will produce a sequence diagram:

```mermaid
sequenceDiagram
Alice ->> Bob: Hello Bob, how are you?
Bob-->>John: How about you John?
Bob--x Alice: I am good thanks!
Bob-x John: I am good thanks!
Note right of John: Bob thinks a long<br/>long time, so long<br/>that the text does<br/>not fit on a row.

Bob-->Alice: Checking with John...
Alice->John: Yes... John, how are you?
```

And this will produce a flow chart:

```mermaid
graph LR
A[Square Rect] -- Link text --> B((Circle))
A --> C(Round Rect)
B --> D{Rhombus}
C --> D
```

<!--stackedit_data:
eyJoaXN0b3J5IjpbODk4MDI0NDAzLDE0MjI1NDQ5NTgsMTkxMT
I4MDAyNSw0OTM2NzE4MDYsLTU2OTIyODE3OSw0NDA1MzI3MF19

-->