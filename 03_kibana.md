# Kibana Setup

After running the installation, wait a few minutes for Fluentd to collect logs and send those over to Elasticsearch.  Elasticsearch will automatically index this information automatically.  To view this information we will be setting up Kibana.

1.  Open Kibana by opening this url, replacing with your own cluster-name: `https://kibana-openshift-logging.apps.<cluster-name>`

2.  For each standard user, after logging in, create an index pattern named `app` and use the `@timestamp` time field to view the app container logs

3.  For each admin user, after logging in, create an index pattern named `app`, `infra`, and optionally `audit` indices with the `@timestamp` field

4.  Create Kibana Visualizations from the newly created index patterns

If you want to later set up indexes, click `Management` in the left nav and then `Index Patterns`.
