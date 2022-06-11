## SDN 第二代 - P4switch
- 特點 : 可自行定義封包
## V1Model P4 Architecture
### ![20220530-1](/img/20220530-1.jpg)
- Parser : 了解封包欄位的資訊 ,例如拆解成 
```
[ethernet]              [ethernet]
                or      [ip]
[payload]               [payload]
```
- Deparser : 把之前拆解的內容還原回來
```
[ethernet][payload] or [ethernet][ip][payload]
```
- ingress match + action pipeline : (入口)符合規則 ,做甚麼動作
- traffic manager : 加入 Queuing 、Replication , 或其他
- engress match + action pipeline : (出口)符合規則 ,做甚麼動作
### ![20220530-2](/img/20220530-2.jpg)
### ![20220530-1=3](/img/20220530-3.jpg)
### 規則匹配
### ![20220530-4](/img/20220530-4.jpg)
- key : 規則
- exact : 完全匹配
## ex1.基本架構
### ![20220530-5](/img/20220530-5.jpg)
### cd /home/user/p4-test/1
### p4run
### 另一個終端機 simple_switch_CLI --thrift-port 9090
### 查看規則
### RuntimeCmd: table_dump phy_forward
### 刪除規則
### RuntimeCmd: table_delete phy_forward [ ? ]
### 添加規則
### RuntimeCmd: table_add phy_forward forward 1 => 2 
### RuntimeCmd: table_add phy_forward forward 2 => 1
### 其他新增規則方法 : # simple_switch_CLI --thrift-port 9090 < cmd.txt
## ex2.
### ![20220530-6](/img/20220530-6.jpg)
- p4app.json
```json
{
  "program": "basic.p4",
  "switch": "simple_switch",
  "compiler": "p4c",
  "options": "--target bmv2 --arch v1model --std p4-16",
  "switch_cli": "simple_switch_CLI",
  "cli": true,
  "pcap_dump": false,
  "enable_log": false,
  "topo_module": {
    "file_path": "",
    "module_name": "p4utils.mininetlib.apptopo",
    "object_name": "AppTopoStrategies"
  },
  "controller_module": null,
  "topodb_module": {
    "file_path": "",
    "module_name": "p4utils.utils.topology",
    "object_name": "Topology"
  },
  "mininet_module": {
    "file_path": "",
    "module_name": "p4utils.mininetlib.p4net",
    "object_name": "P4Mininet"
  },
  "topology": {
    "links": [["h1", "s1"], ["h2", "s2"], ["s1", "s2"]],
    "hosts": {
      "h1": {
      },
      "h2": {
      }
    },
    "switches": {
      "s1": {
        "cli_input": "cmd.txt",
        "program": "basic.p4"
      },
      "s2": {
        "cli_input": "cmd.txt",
        "program": "basic.p4"
      }
    }
  }
}
```
### 重新設置網路
### mininet> h2 ifconfig h2-eth0 0
### mininet> h2 ip addr add 10.0.1.2/24 brd + dev h2-eth0
### mininet> h1 ping h2 -c 3
### or 直接改 p4app.json
```json
{
  "program": "basic.p4",
  "switch": "simple_switch",
  "compiler": "p4c",
  "options": "--target bmv2 --arch v1model --std p4-16",
  "switch_cli": "simple_switch_CLI",
  "cli": true,
  "pcap_dump": false,
  "enable_log": false,
  "topo_module": {
    "file_path": "",
    "module_name": "p4utils.mininetlib.apptopo",
    "object_name": "AppTopoStrategies"
  },
  "controller_module": null,
  "topodb_module": {
    "file_path": "",
    "module_name": "p4utils.utils.topology",
    "object_name": "Topology"
  },
  "mininet_module": {
    "file_path": "",
    "module_name": "p4utils.mininetlib.p4net",
    "object_name": "P4Mininet"
  },
  "topology": {
    "assignment_strategy": "l2",  
    "links": [["h1", "s1"], ["h2", "s2"], ["s1", "s2"]],
    "hosts": {
      "h1": {
      },
      "h2": {
      }
    },
    "switches": {
      "s1": {
        "cli_input": "cmd.txt",
        "program": "basic.p4"
      },
      "s2": {
        "cli_input": "cmd.txt",
        "program": "basic.p4"
      }
    }
  }
}
```
### ex3.
### ![20220530-7](/img/20220530-7.jpg)
### 規則 : 
```
table_add mac_forward forward 00:00:0a:00:01:01 => 1
table_add mac_forward forward 00:00:0a:00:01:02 => 2
```
### cd /home/user/p4-test/2
### p4run
### mininet> h1 ping h2 -c 3
## ex4.
### ![20220530-8](/img/20220530-8.jpg)
### 實現 h1 h2 h3 皆可互 ping
- p4app.json
```json
{
  "program": "basic.p4",
  "switch": "simple_switch",
  "compiler": "p4c",
  "options": "--target bmv2 --arch v1model --std p4-16",
  "switch_cli": "simple_switch_CLI",
  "cli": true,
  "pcap_dump": true,
  "enable_log": true,
  "topo_module": {
    "file_path": "",
    "module_name": "p4utils.mininetlib.apptopo",
    "object_name": "AppTopoStrategies"
  },
  "controller_module": null,
  "topodb_module": {
    "file_path": "",
    "module_name": "p4utils.utils.topology",
    "object_name": "Topology"
  },
  "mininet_module": {
    "file_path": "",
    "module_name": "p4utils.mininetlib.p4net",
    "object_name": "P4Mininet"
  },
  "topology": {
    "links": [["h1", "s1"], ["h2", "s1"], ["h3", "s1"]],
    "hosts": {
      "h1": {
      },
      "h2": {
      },
      "h3": {
      }
    },
    "switches": {
      "s1": {
        "cli_input": "cmd.txt",
        "program": "basic.p4"
      }
    }
  }
}

```
### p4run
### mininet> h1 ping h2 -c 3