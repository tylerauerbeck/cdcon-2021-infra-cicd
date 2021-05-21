## Local Break (this doesn't work on M1 machines right now because the bind image isn't build for arm)

TODO: Ansible to come soon to automate all of this

#Set to either docker or podman
#export CT=podman
#export CT=docker

$CT network create rht-labs.io --subnet=172.20.0.0/16
$CT run -d --name bind --rm --publish 5354:53/tcp --publish 5354:53/udp --publish 10000:10000/tcp --net=rht-labs.io --ip=172.20.0.2 sameersbn/bind:latest
$CT run -d --name host-1 --rm --net=rht-labs.io --ip=172.20.0.3 --dns=172.20.0.2 centos:8 /usr/sbin/init
podman run --net=rht-labs.io --ip=172.20.0.4 --dns=172.20.0.2 -d -p 8080:8080 hashicorp/http-echo -listen=:8080 -text="hello world"

https://localhost:10000 -> root:password -> servers -> bind dns server -> create m zone -> rht-labs.io (webmin@rht-labs.io) -> apply configuration/zone (top right corner) -> create text record (hello.rht-labs.io) -> create address (host-1.rht-labs.io: 172.20.0.3, myapp.rht-labs.io: 172.20.0.4) -> apply configuration/zone

$CT exec -it host1 bash
curl myapp.rht-labs.io:8080 ("hello world!")

back to webmin -> make some silly change (apply config) -> watch it break

## Cluster Magic (CRC)

TODO: Ansible to come soon to automate this
TODO: We should make sure to provide the magic to make this work on both OCP & K8S (kind)

crc start (24 GB mem, 6 CPU)

oc new-project cnv-tekton
oc create sa pipeline-sa -n cnv-tekton
oc create clusterrolebinding infra-cicd-sp --clusterrole=self-provisioner --serviceaccount=cnv-tekton:pipeline-sa

Go to console and install openshift-virtualization and openshift-pipelines
