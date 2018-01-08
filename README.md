**__Note__**: Because I'm running my own servers for serveral years, all data and repositories are hosted there
at <https://git.ypbind.de/cgit/prometheus-icinga2stats-exporter/about/>

---

Export internal Icinga2 statistics to Prometheus
================================================

## Preface
[Icinga2](https://www.icinga.com/products/icinga-2/) exposes internal statistics via it's [REST API](https://www.icinga.com/docs/icinga2/latest/doc/12-icinga2-api/)

The purpose of this program is to fetch these statistics and make them available for [Prometheus](https://prometheus.io).

*Note*: The internal statistics are also reported as `CheckResult` objects. This allows  [prometheus-icinga2perfdata-exporter](https://git.ypbind.de/cgit/prometheus-icinga2perfdata-exporter/)
to export the statistics in combination with the performance data reported by regular Icinga2 checks.

---

## Method of operation
[prometheus-icinga2stats-exporter](https://git.ypbind.de/cgit/prometheus-icinga2stats-exporter/) runs as a process in the foreground (suitable for running as [systemd](https://www.freedesktop.org/wiki/Software/systemd/))
service and query the [Icinga2 status](https://www.icinga.com/docs/icinga2/latest/doc/12-icinga2-api/#status-and-statistics) endpoint at regular intervals.

The reported statistics are parsed and stored in memory. [prometheus-icinga2stats-exporter](https://git.ypbind.de/cgit/prometheus-icinga2stats-exporter/) listens (default port 19998 on all interfaces) for
HTTP connections and reports the collected data when `/metrics` is queried by the Prometheus scraper.

---

## Build requirements
### Go!
Obviously because the exporter has been written in [Go](https://golang.org/)

### Go package - github.com/go-ini/ini
Parsing the INI format of configuration is done using the [github.com/go-ini/ini](https://github.com/go-ini/ini) package

---

## Configuration
### Icinga2 permissions
[prometheus-icinga2stats-exporter](https://git.ypbind.de/cgit/prometheus-icinga2stats-exporter/) collects the internal statistics by connecting to the Icinga2 status endpoint of the Icinga2 REST API.

Therefore the only required permission for the API user used to connect is `status/query`.

### prometheus-icinga2stats-exporter configuration
At the moment two sections are recognized.

The mandatory `icinga2` section specify the connection and authentication parameters to connect to the Icinga2 API and the
optional `exporter` section allows for overriding the instance name and for expiration timer for collected performance data.

#### Section `icinga2`
Mandatory is the `[icinga2]` section as it contains the connection and authentication options required to connect to the EventStream API of Icinga2.

Parameters are:

  * `host` - host name (or IP address) of the Icinga2 instance to connect to
  * `port` - port number to connect to; _Default:_ 5665 (note: this *must* be a number, service names are not supported)
  * `method` - authentication method, this can be either
    * `user` - authenticate with a user/password combination using HTTP basic authentication
    * `cert` - authenticate using a X.509 certificate
  * `user` - username to use for authentication if `method` is set to `user`
  * `password`- password to use for authentication if method is set to `password`
  * `cert_file` - absolute path to the public key of the X.509 certificate used for authentication if `method` is set to `cert`
  * `key_file` - absolute path to the private key of the X.509 certificate used for authentication if `method` is set to `cert` (note: due to the lack of Go to load encrypted keys only _unencrypted_ X.509 keys are supported)
  * `ca_cert` - absolute path to the CA file for the Icinga2 service
  * `insecure_ssl` - skip verification if the hostname provided by the certificate of the Icinga2 instance matches the `host` name, valid values are `true` or `false` (default)

#### Section `exporter`
The optional section `exporter` supports only one option:

  * `instance` - override the name of the reporting instance (label `instance`), the default is the `host:port` of the Icinga2 instance connected to

#### Sample configuration

An example configuration file can be found in the `etc` directory of this package.

---

## Command line parameters
prometheus-icinga2stats-exporter supports the following command line parameters:

  * `-config-file` - location of the configuration file, default: `/etc/prometheus/prometheus-icinga2stats-exporter.ini`
  * `-help` - displays a short help text
  * `-listen-address` - address to listen for requests, default: `:19998`

---

## License
This program is licenses under [GLPv3](http://www.gnu.org/copyleft/gpl.html).


