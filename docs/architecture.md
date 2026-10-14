# Lab Architecture

Three Ubuntu VMs run in VMware. The monitor routes between separate client and server networks and inspects their traffic with Suricata.

```mermaid
%%{init: {"theme": "base", "look": "classic", "flowchart": {"nodeSpacing": 40, "rankSpacing": 45, "padding": 30, "diagramPadding": 25, "subGraphTitleMargin": {"top": 16, "bottom": 20}, "curve": "basis"}, "themeVariables": {"fontFamily": "Arial, sans-serif", "fontSize": "16px", "lineColor": "#64748B", "primaryTextColor": "#172033", "clusterBkg": "#F8FAFC", "clusterBorder": "#CBD5E1", "edgeLabelBackground": "#F8FAFC"}}}%%
graph TD
    subgraph CLIENT_NET["Client network · 10.77.10.0/24"]
        CLIENT(["Client · GET / HEAD<br/>10.77.10.10"])
    end

    subgraph MONITOR["Monitor · Ubuntu router + sensor"]
        ROUTER(["IPv4 forwarding + nftables<br/>client0 · 10.77.10.1<br/>server0 · 10.77.20.1"])
        IDS(["Suricata · traffic inspection<br/>GET rule · SID 1000001"])
        LOGS(["EVE JSON alerts<br/>Inspect with jq"])
        ROUTER -.-> IDS
        IDS --> LOGS
    end

    subgraph SERVER_NET["Server network · 10.77.20.0/24"]
        SERVER(["Python HTTP server<br/>10.77.20.20:8080"])
    end

    subgraph MANAGEMENT["Management · SSH"]
        HOST(["Windows host · 172.28.77.1<br/>SSH to 172.28.77.10:22"])
    end

    CLIENT <--> ROUTER
    ROUTER <--> SERVER
    HOST -.-> ROUTER

    style CLIENT_NET fill:#F1F5FC,stroke:#CBD5E1,stroke-width:1px,rx:22px,ry:22px
    style MONITOR fill:#F8F6FC,stroke:#CBD5E1,stroke-width:1px,rx:22px,ry:22px
    style SERVER_NET fill:#F1F5FC,stroke:#CBD5E1,stroke-width:1px,rx:22px,ry:22px
    style MANAGEMENT fill:#F8FAFC,stroke:#CBD5E1,stroke-width:1px,rx:22px,ry:22px

    classDef endpoint fill:#E8F0FC,stroke:#26364A,stroke-width:1.5px,color:#172033;
    classDef routing fill:#EDE9F8,stroke:#26364A,stroke-width:1.5px,color:#172033;
    classDef detection fill:#E2F3EB,stroke:#26364A,stroke-width:1.5px,color:#172033;
    classDef management fill:#F1F3F6,stroke:#26364A,stroke-width:1.5px,color:#172033;
    class CLIENT,SERVER endpoint;
    class ROUTER routing;
    class IDS,LOGS detection;
    class HOST management;
```

**Reading the diagram:** The two-way arrows carry HTTP requests and responses. Dashed arrows show the separate management SSH connection and traffic inspection by Suricata on `client0`. The arrow from Suricata to EVE JSON represents log output.

| System/interface | Address | Purpose |
|---|---|---|
| Client | 10.77.10.10/24 | Generates test requests |
| Monitor: client0 | 10.77.10.1/24 | Client-side gateway and capture interface |
| Monitor: server0 | 10.77.20.1/24 | Server-side gateway |
| Server | 10.77.20.20/24 | Python HTTP server on port 8080 |
| Monitor: mgmt0 | 172.28.77.10/24 | Host management connection |
| Windows host management address | 172.28.77.1 | SSH access to the monitor |

The monitor's `update0` interface is down in the exported snapshot. Its main routing table contains connected routes for the three lab/management networks and no default route. IPv4 forwarding is enabled.

Suricata observes the traffic; requests do not pass through it as a blocking proxy.

## Firewall policy

The exported nftables ruleset uses default-drop input and forwarding chains and an accept output policy. It permits traffic between the client and server subnets in both directions, and permits management SSH from `172.28.77.1`, along with the other input exceptions shown in the ruleset.

The forwarding policy permits general traffic between those two subnets; it is not limited to HTTP port 8080. Segmentation provides a routing/inspection boundary, not isolation between the client and server.

## Configuration evidence

- [Monitor interfaces](../configs/network/monitor-interfaces.txt)
- [Monitor routes](../configs/network/monitor-routes.txt)
- [IPv4 forwarding](../configs/network/monitor-ip-forwarding.txt)
- [nftables ruleset](../configs/nftables/monitor-ruleset.nft)
- [Suricata rule](../configs/suricata/local.rules)

These files are observed configuration/state exports. The nftables file includes counters from collection time. Persistent per-VM network configuration and the Suricata YAML/service configuration have not yet been included, so this is not a complete automated rebuild bundle.
