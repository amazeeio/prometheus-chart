# amazee.io Prometheus Chart

This Prometheus Helm Chart contains customizations for OpenShift and specifically amazee.io OpenShift Clusters.

# Usage

1. Update `values.yml`
2. Run

        helm --tiller-namespace tiller upgrade --install prometheus-test coreos/kube-prometheus --namespace prometheus-test -f values.yaml

# Installation

As OpenShift has much higher security standards than a regular Kubernetes, the installation is a bit tricky:

## Dependency: Tiller

1. If helm/tiller is not installed make sure tiller server is installed by following https://blog.openshift.com/getting-started-helm-openshift/

> Note: tiller requires admin cluster role. Add it with:

        oc adm policy add-cluster-role-to-user cluster-admin system:serviceaccount:tiller:tiller

## Dependency: Prometheus Operator

1. Create a new OpenShift Project (here we're using `prometheus-test` but it can be anything)

        oc new-project prometheus-test

2. Allow the project to use all nodes, not just only the compute nodes (needed for the NodeExporter)

        oc annotate namespace prometheus-test openshift.io/node-selector=

3. Install the Prometheus operator.

        helm --tiller-namespace tiller install coreos/prometheus-operator --name prometheus-operator-test --namespace prometheus-test

4. The Prometheus operator needs cluster admin rights, so provide that:

        oc adm policy add-cluster-role-to-user cluster-admin -z prometheus-operator-test --namespace prometheus-test

## Install kube-prometheus

6. Install kube-prometheus (this will actually install Prometheus/Grafana/Alertmanager):

        helm --tiller-namespace tiller upgrade --install prometheus-test coreos/kube-prometheus --namespace prometheus-test -f values.yaml

7. The above helm upgrade/install might fail or the pods are not comming up, that's because they need additional permissions:

        oc adm policy add-scc-to-user anyuid -z prometheus-test-grafana --namespace prometheus-test
        oc adm policy add-scc-to-user anyuid -z prometheus-test-exporter-kube-state --namespace prometheus-test
        oc adm policy add-scc-to-user anyuid -z default --namespace prometheus-test
        oc adm policy add-scc-to-user anyuid -z prometheus-test --namespace prometheus-test
        oc adm policy add-scc-to-user privileged -z prometheus-test-exporter-node --namespace prometheus-test

8. Run the Upgrade/Install for kube-prometheus again:

        helm --tiller-namespace tiller upgrade --install prometheus-test coreos/kube-prometheus --namespace prometheus-test -f values.yaml

9. Create Routes to Services for Grafana, Prometheus and Alertmanager via the OpenShift UI

# Mysql exporter

1. Create "mysql-exporter" user on the DB instance and grant access
        GRANT PROCESS, REPLICATION CLIENT, SELECT ON *.* TO 'mysql-exporter'@'%' identified by 'password' with grant option;

2. Deploy mysql-exporter into "prometheus-prod" namespace:
        helm --tiller-namespace tiller upgrade --install mysql-exporter stable/prometheus-mysql-exporter --set datasource="mysql-exporter:password@(mysql-hostname:3306)/" --namespace prometheus-prod

3. Configure Prometheus to scrape the above. See example config in `values.yaml`.