# amazee.io Prometheus Chart

This Prometheus Helm Chart contains customizations for OpenShift and specifically amazee.io OpenShift Clusters.

# Usage

1. Update `values.yml`
2. Run

        helm --tiller-namespace tiller upgrade --install prometheus-test coreos/kube-prometheus --namespace prometheus-test -f values.yaml

# Installation

As OpenShift has much higher security standards than a regular Kubernetes, the installation is a bit tricky:

1. Create a new OpenShift Project (here we're using `prometheus-test` but it can be anything)

        oc new project prometheus-test

2. Allow the project to use all nodes, not just only the compute nodes (needed for the NodeExporter)

        oc annotate project prometheus-test 'openshift.io/node-selector=""'

3. Install the prometheus operator

        helm --tiller-namespace tiller install coreos/prometheus-operator --name prometheus-operator-test --namespace prometheus-test

4. The prometheus operator needs cluster admin rights, so provide that:

        oc adm policy add-cluster-role-to-user cluster-admin -z prometheus-operator-test --namespace prometheus-test

5. Install kube-prometheus (this will actually install prometheus):

        helm --tiller-namespace tiller upgrade --install prometheus-test coreos/kube-prometheus --namespace prometheus-test -f values.yaml

6. The above helm upgrade/install might fail or the pods are not comming up, that's because they need additional permissions:

        oc adm policy add-scc-to-user anyuid -z prometheus-test-grafana --namespace prometheus-test
        oc adm policy add-scc-to-user anyuid -z prometheus-test-exporter-kube-state --namespace prometheus-test
        oc adm policy add-scc-to-user anyuid -z default --namespace prometheus-test
        oc adm policy add-scc-to-user anyuid -z prometheus-test --namespace prometheus-test
        oc adm policy add-scc-to-user privileged -z prometheus-test-exporter-node --namespace prometheus-test

7. Run the Upgrade/Install for kube-prometheus again:

        helm --tiller-namespace tiller upgrade --install prometheus-test coreos/kube-prometheus --namespace prometheus-test -f values.yaml


# Mysql exporter

1. Create "mysql-exporter" user on the DB instance and grant access
        GRANT PROCESS, REPLICATION CLIENT, SELECT ON *.* TO 'mysql-exporter'@'%' identified by 'password' with grant option;

2. Deploy mysql-exporter into "prometheus-prod" namespace:
        helm --tiller-namespace tiller upgrade --install mysql-exporter stable/prometheus-mysql-exporter --set datasource="mysql-exporter:password@(mysql-hostname:3306)/" --namespace prometheus-prod

3. Configure Prometheus to scrape the above