# Jackett Helm

This is a helm chart for [Jackett](https://github.com/Jackett/Jackett), and includes an optional [Gluetun](https://github.com/qdm12/gluetun) VPN proxy.

https://github.com/sypticus/jackett-helm.git

## Installing the Chart.

```console
git clone https://github.com/sypticus/jackett-helm.git
helm install jackett ./jackett-helm/jackett -n torrent --create-namespace`
```

You should then be able to access the dashboard.

```console
POD=$(kubectl get pods -l app.kubernetes.io/instance=jackett  -o jsonpath="{.items[0].metadata.name}" -n torrent)
kubectl -n proxy port-forward $POD 9117
```
In another window open http://localhost:9117

## Persistence
By default the config and download (blackhole) folders are dropped at each deploy. Turn on `persistence.enabled` to allow storage.
If you already have a PVC you would like to use, set it in values.yaml, otherwise a new PVC will be created for you and used.


## Gluetun
If you would like to start up the Gluetun sidecar and hide your IP behind a VPN, you can pass in configuration.

```console
helm install jackett jackett/ --set gluetun.enabled=true --set gluetun.openvpnUser=$YOUR_MULLVAD_ACCOUNT_NUM

```
The container defaults to a Mullvad openVPN, but you can change to any number of providers.
You will need to set up credentials for either openVPN or Wireguard, so you can also define secrets which contain VPN creds, so you don't have to commit those.

You should see confirmation of your new IP in the logs on the container.


## Deploying for Real

In order to set the correct values for your instance, create a values.yaml to override the defaults. 

```console
helm install jackett ./jackett-helm -f my_values.yaml -n torrent
```


## Uninstalling the Chart

```console
helm delete jacket
```

## Configuration

Documentation of configs can be found on the Jackett [Github](https://github.com/Jackett/Jackett)
Further info can be found in the charts included [values.yaml](https://github.com/sypticus/jackett-helm/blob/main/jackett/values.yaml)


The following are the most of the relevant config values which can be set in the helm chart.
All other config values can be passed in as environmental variables using the `extraParams` field, or `gluetun.extraParams`
For Gluetun providers and parameters, see the wiki at https://github.com/qdm12/gluetun-wiki


| Parameter                      | Description                                                            | Default                       |
|--------------------------------|------------------------------------------------------------------------|-------------------------------|
| `image.repository`             | Image repository                                                       | `lscr.io/linuxserver/jackett` |
| `image.tag`                    | Image tag                                                              | `0.22.1211`                   |
| `image.pullPolicy`             | Image pull policy                                                      | `IfNotPresent`                |
| `service.type`                 | Kubernetes service type                                                | `ClusterIP`                   |
| `service.port`                 | Port to be used for http calls.                                        | `9117`                        |
| `service.loadBalancerIP`       | If type is LoadBalancer, set the IP                                    |                               |
| `persistence.enabled`          | Use a PVC to save config and downloads                                 | `false`                       |
| `persistence.existingClaim`    | An existing VPC to use for storage                                     |                               |
| `persistence.storageClassName` | Storage class to use if creating a new PVC                             |                               |
| `extraParams`                  | Extra params to be passed into the Jackett container                   |                               |
| `gluetun.enabled`              | Startup a Gluetun sidecar to proxy calls through a VPN                 | `false`                       |
| `gluetun.vpnType`              | wireguard or openvpn                                                   | `openvpn`                     |
| `gluetun.openvpnSecret`        | k8s secret with openvpn credentials (openvpnuser='', openvpnpasswd='') |                               |
| `gluetun.openvpnUser`          | inline openvpn username                                                |                               |
| `gluetun.openvpnPassword`      | inline openvpn password                                                |                               |
| `gluetun.wireguardAddresses`   | Addresses given by your Wireguard provider                             |                               |
| `gluetun.wireguardSecret`      | k8s secret with wireguard private key (wgprivatekey='')                |                               |
| `gluetun.wireguardKey`         | inline wireguard private key                                           |                               |
| `gluetun.dnsAddress`           | Preferred DNS address                                                  | `10.64.0.1`                   |
| `gluetun.providerParams`       | any extra required env vars for provider                               | `[]`                          |
