# Helmfile

The aim of this repository is to provide a collection of Helmfile configurations that make up
the barebones of my k3 cluster. After a weekend when I stupidly losts the k3s master nodes token
thus making the existing MySQL database useless, I decided to start from scratch that would capture
the configurations I need in place for a full disaster recovery.

This is the fruit of that labor.

## Setup

Getting this tool that's still in beta was not as straightforward as I thought initially.
I ended up wrapping the dockerfile in a alias to make it easier to use.

This is the config in `~/.config/fish/config.fish`

```bash
function helmfile
    docker run --rm --net=host -v "$HOME/.kube:/helm/.kube" -v "$HOME/.config/helm:/helm/.config/helm" -v "$PWD:/wd" --workdir /wd ghcr.io/helmfile/helmfile:v1.0.0-rc.7 helmfile $argv --file /wd/helmfile.yaml
end
```

## Project Structure

The structure of the chart files is as follows:


<code-block lang="mermaid">
graph TD
    Root["/"]
    Root --> home_assistant["home_assistant"]
    home_assistant --> home_assistant_values["values.yaml"]
    Root --> metallb["metallb"]
    metallb --> metallb_values["values.yaml"]
    Root --> longhorn["longhorn"]
    longhorn --> longhorn_values["values.yaml"]
    Root --> prometheus["prometheus-operator"]
    Root --> cert_manager["cert_manager"]
    cert_manager --> cert_manager_values["values.yaml"]
    prometheus --> prometheus-operator_values["values.yaml"]
    Root --> helmfile["helmfile.yaml"]
</code-block>

Root
    helmfile.yaml


```yaml
# Website for Helmfile: https://helmfile.readthedocs.io/en/latest/
repositories:
  - name: metallb
    url: https://metallb.github.io/metallb
  - name: jetstack
    url: https://charts.jetstack.io
  - name: longhorn
    url: https://charts.longhorn.io
  - name: prometheus-community
    url: https://prometheus-community.github.io/helm-charts
  - name: kube-prometheus-stack
    url: https://prometheus-community.github.io/helm-charts

releases:
# Base Platform
  - name: metallb
    namespace: metallb-system
    chart: metallb/metallb # Load Balancer solution for bare metal Kubernetes
    version: 0.14.8
    values:
      - ./metallb/values.yaml
    hooks:
# This is commented out as since hooks are not aware of the state of helm deployment, then it can lead
# to timing issues. As such, knowing about hooks is a good thing however I might be able to acheive the
# same thing by breaking out these scripts into their own helm chart.
  

#       - events: ["postsync"]
#         showlogs: true
#         command: "sh"
#         args: ["-c", "sleep 30 && kubectl apply -f ./metallb/ipaddresspool.yaml -n metallb-system"]
#       - events: ["postsync"]
#         showlogs: true
#         command: "sh"
#         args: ["-c", "sleep 30 && kubectl apply -f ./metallb/l2advertisement.yaml -n metallb-system"]
  - name: longhorn # Storage
    namespace: longhorn-system
    chart: longhorn/longhorn
    version: 1.6.3

  - name: prometheus-operator
    namespace: monitoring
    chart: prometheus-community/kube-prometheus-stack
    values:
      - ./prometheus-operator/values.yaml

  - name: cert-manager
    namespace: cert-manager
    chart: jetstack/cert-manager
    version: v1.16.1
    set: installCRDs=true
```

<note type="tip">
Post deployment Hooks are not worth the added complexity. I can achieve the same thing by breaking out the scripts into their own helm chart.
</note>