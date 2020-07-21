# Monitoring

* Hosted on: `monitoring.infra.rust-lang.org` (behind the bastion -- [how to connect][bastion-connect])
* Maintainers: [pietroalbini], infra team
* Public URL: [grafana.rust-lang.org](https://grafana.rust-lang.org)
* [Ansible playbook][ansible-playbook] to deploy this server.
* [Instance metrics][grafana-instance] (only available to infra team members).

## Service configuration

Our monitoring service is composed of three parts: [Prometheus] to scrape,
collect and monitor metrics, [Alertmanager] to dispatch the alerts generated by
Prometheus, and [Grafana] to display the metrics. All the parts are configured
through [Ansible].

The metrics are not backed up, as Prometheus purges them after 7 days anyway,
but the Grafana dashboards are stored in a PostgreSQL database, which is backed
up with [restic] in the `rust-backups` bucket (`monitoring` subdirectory). The
password to decrypt the backups is in 1password.

## Common maintenance procedures

### Scrape a new metrics source

Prometheus works by periodically scraping a list of HTTP endpoints for metrics,
written [in its custom format][metrics-format]. In our configuration the list
is located in the `prometheus_scrape` section of the
`ansible/playbooks/monitoring.yml` file in the [simpleinfra] repository.

To add a new metrics source, add your endpoint to an existing job or, if the
metrics you're scraping are not related to any other job, a new one. The
endpoint must be reachable from the monitoring instance. You can read the
[Prometheus documentation][prometheus-scrape] to find all the available
options.

### Create a new alert

Alerts are generated by Prometheus every time a custom rule defined in its
configuration evaluates to true. In our configuration the list of rules is
located in the `prometheus_rule_groups` section of the
`ansible/playbooks/monitoring.yml` file in the [simpleinfra] repository.

To add a new alert you need to create an alerting rule either in an existing
group or a new one. The full list of options is available in the [Prometheus
documentation][prometheus-alert].

### Add permissions to a user

There are two steps needed to grant access to [our Grafana
instance][grafana-ours] to an user.

First of all, to enable the user to log into the instance with their GitHub
account they need to be a member of a team authorized to log in. The list of
teams is defined in the `grafana_github_teams` section of the
`ansible/playbooks/monitoring.yml` file in the [simpleinfra] repository, and it
contains a list of GitHub team IDs. To fetch an ID you can run this command:

```
curl -H "Authorization: token $GITHUB_TOKEN" https://api.github.com/orgs/<ORG>/teams/<NAME> | jq .id
```

Once the user is a member of a team authorized to log in they will
automatically be added to the main Grafana organization with "viewer"
permissions. For infrastructure team members that needs to be changed to
"admin" (in the "Configuration" -> "Users"), otherwise leave it as viewer.

By default a viewer only has access to the unrestricted dashboards. To grant
access to other dashboards you'll need to add them to a team (in the
"Configuration" -> "Teams" page). It's also possible to grant admin privileges
to the whole Grafana instance in the "Server Admin" -> "Users" ->
"`<username>`" page. Do not grant those permissions except to trusted infra
team members.

## Additional resources

* [Prometheus documentation][prometheus-docs]
* [Grafana documentation][grafana-docs]

[bastion-connect]: https://github.com/rust-lang/infra-team/blob/master/docs/hosts/bastion.md#logging-into-servers-through-the-bastion
[pietroalbini]: https://github.com/pietroalbini
[ansible-playbook]: https://github.com/rust-lang/simpleinfra/blob/master/ansible/playbooks/monitoring.yml
[grafana-instance]: https://grafana.rust-lang.org/d/rpXrFfKWz/instance-metrics?orgId=1&var-instance=monitoring.infra.rust-lang.org:9100
[Prometheus]: https://prometheus.io
[Alertmanager]: https://prometheus.io/docs/alerting/alertmanager/
[Grafana]: https://grafana.com
[Ansible]: https://github.com/rust-lang/simpleinfra/tree/master/ansible
[restic]: https://restic.net
[metrics-format]: https://prometheus.io/docs/instrumenting/exposition_formats/
[simpleinfra]: https://github.com/rust-lang/simpleinfra
[prometheus-scrape]: https://prometheus.io/docs/prometheus/latest/configuration/configuration/#scrape_config
[prometheus-alert]: https://prometheus.io/docs/prometheus/latest/configuration/alerting_rules/
[grafana-ours]: https://grafana.rust-lang.org
[prometheus-docs]: https://prometheus.io/docs/introduction/overview/
[grafana-docs]: https://grafana.com/docs/