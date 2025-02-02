**1.下载Sing-box**
```
wget https://github.com/SagerNet/sing-box/releases/download/v1.10.3/sing-box-1.10.3-linux-amd64.tar.gz
tar -xzvf sing-box-1.10.3-linux-amd64.tar.gz  && cd sing-box-1.10.3-linux-amd64
```

**2.修改下面的配置文件,保存为config.json**
```
{
    "log": {
        "disabled": false,
        "level": "warn",
        "timestamp": true
    },
  "dns": {
    "servers": [
      {
        "tag": "remote",
        "address": "tcp://8.8.8.8",
        "strategy": "ipv4_only",
        "detour": "cfg-hysteria2-out"
      },
      {
        "tag": "local",
        "address": "223.5.5.5",
        "strategy": "ipv4_only",
        "detour": "direct"
      },
      {
        "tag": "block",
        "address": "rcode://success"
      },
      {
        "tag": "local_local",
        "address": "223.5.5.5",
        "detour": "direct"
      }
    ],
    "rules": [
      {
        "server": "remote",
        "clash_mode": "Global"
      },
      {
        "server": "local_local",
        "clash_mode": "Direct"
      },
      {
        "server": "local",
        "rule_set": [
          "geoip-cn",
          "geosite-geolocation-cn"
        ]
      },
      {
        "server": "block",
        "rule_set": [
          "cfg-ad-rule"
        ]
      }
    ],
    "final": "remote"
  },
    "inbounds": [
        {
            "type": "tun",
            "tag": "tun-in",
            "interface_name": "singtun0",
            "address": ["172.19.0.1/30"],
            "mtu": 9000,
            "gso": false,
            "auto_route": true,
            "udp_timeout": "300s",
            "stack": "system",
            "sniff": true,
            "sniff_override_destination": false
        }
    ],
    "outbounds": [
        {
            "type": "direct",
            "tag": "direct-out",
            "routing_mark": 100
        },
        {
            "type": "block",
            "tag": "block-out"
        },
        {
            "type": "dns",
            "tag": "dns-out"
        },
        {
            "type": "hysteria2",
            "tag": "cfg-hysteria2-out",
            "routing_mark": 100,
            "server": "服务器IP",
            "server_port": 端口号,
            "password": "密码",
            "tls": {
                "enabled": true,
                "server_name": "bing.com",
                "insecure": true
            },
            "domain_strategy": "prefer_ipv4"
        },
        {
            "type": "selector",
            "tag": "select-out",
            "outbounds": [
                "cfg-hysteria2-out"
            ],
            "default": "cfg-hysteria2-out",
            "interrupt_exist_connections": true
        }
    ],
    "route": {
        "rules": [
            {
                "inbound": "dns-in",
                "outbound": "dns-out"
            },
            {
                "type": "logical",
                "mode": "or",
                "rules": [
                    {
                        "protocol": "dns"
                    },
                    {
                        "port": 53
                    }
                ],
                "outbound": "dns-out"
            },
            {
                "ip_is_private": true,
                "outbound": "direct-out"
            },
            {
                "clash_mode": "Direct",
                "outbound": "direct-out"
            },
            {
                "clash_mode": "Global",
                "outbound": "select-out"
            },
            {
                "type": "logical",
                "mode": "or",
                "rules": [
                    {
                        "port": 853
                    },
                    {
                        "network": "udp",
                        "port": 443
                    },
                    {
                        "protocol": "stun"
                    }
                ],
                "outbound": "block-out"
            },
            {
                "inbound": [
                    "mixed-in"
                ],
                "outbound": "cfg-hysteria2-out"
            }
        ],
        "rule_set": [
			{
				"type": "remote",
				"tag": "geosite-geolocation-cn",
				"format": "binary",
				"url": "https://raw.githubusercontent.com/SagerNet/sing-geosite/rule-set/geosite-geolocation-cn.srs",
				"download_detour": "select-out"
			},
			{
				"type": "remote",
				"tag": "geosite-geolocation-!cn",
				"format": "binary",
				"url": "https://raw.githubusercontent.com/SagerNet/sing-geosite/rule-set/geosite-geolocation-!cn.srs",
				"download_detour": "select-out"
			},
			{
				"type": "remote",
				"tag": "geoip-cn",
				"format": "binary",
				"url": "https://raw.githubusercontent.com/SagerNet/sing-geoip/rule-set/geoip-cn.srs",
				"download_detour": "select-out"
			},
			{
				"type": "remote",
				"tag": "cfg-ad-rule",
				"format": "binary",
				"url": "https://github.com/TG-Twilight/AWAvenue-Ads-Rule/raw/main/Filters/AWAvenue-Ads-Rule-Singbox.srs",
				"download_detour": "select-out"
			}		
        ],
        "auto_detect_interface": true,
        "final": "select-out"
    },
    "experimental": {
        "cache_file": {
            "enabled": true
        },
        "clash_api": {
            "external_controller": "0.0.0.0:9090",
            "external_ui": "ui",
            "external_ui_download_url": "https://github.com/MetaCubeX/Yacd-meta/archive/gh-pages.zip",
            "external_ui_download_detour": "select-out",
			"secret": "访问密码",
            "default_mode": "Rule",
			"access_control_allow_origin": [
				"http://127.0.0.1",
				"http://yacd.haishan.me"
			],
			"access_control_allow_private_network": true
        }
    }
}
```
修改其中"outbounds"出站中的服务器的配置信息,"clash_api"面板中的访问密码

**3.运行Sing-box**
```
./sing-box run -c ./config.json
```

**4.使用面板查看Sing-box运行状态**
浏览器输入配置ClashApi的地址端口号输入访问密码,例如:`http://192.168.1.21:9090/ui/#/proxies`
![U120241128022631](https://github.com/user-attachments/assets/7ed00e83-452c-4348-963d-2289d2c01cd0)
![B120241128022631](https://github.com/user-attachments/assets/a14e9099-ca9d-426f-8426-b0ccb87c83e9)


**5.config.json配置文件原始内容**
```
{
    "log": {
        "disabled": false,
        "level": "warn",
        "timestamp": true
    },
    "dns": {
        "servers": [
            {
                "tag": "cfg-Ali-dns",
                "address": "https://223.5.5.5/dns-query?",
                "address_strategy": "ipv4_only",
                "strategy": "ipv4_only",
                "detour": "direct-out"
            },
            {
                "tag": "cfg-google-dns",
                "address": "https://dns.google/dns-query",
                "address_resolver": "cfg-Ali-dns",
                "address_strategy": "ipv4_only",
                "strategy": "ipv4_only",
                "detour": "select-out",
                "client_subnet": "1.0.1.0"
            }
        ],
        "rules": [
            {
                "ip_version": 4,
                "outbound": "any",
                "server": "cfg-Ali-dns"
            },
            {
                "clash_mode": "Direct",
                "server": "cfg-Ali-dns"
            },
            {
                "clash_mode": "Global",
                "server": "cfg-google-dns"
            },
            {
                "rule_set": [
                    "geosite-geolocation-cn"
                ],
                "server": "cfg-Ali-dns"
            },
            {
                "type": "logical",
                "mode": "and",
                "rules": [
                    {
                        "rule_set": "geosite-geolocation-!cn",
                        "invert": true
                    },
                    {
                        "rule_set": "geoip-cn"
                    }
                ],
                "server": "cfg-google-dns"
            }
        ],
        "strategy": "ipv4_only",
        "disable_cache": true,
        "disable_expire": false,
        "independent_cache": false,
        "final": "cfg-google-dns"
    },
    "inbounds": [
        {
            "type": "direct",
            "tag": "dns-in",
            "listen": "::",
            "listen_port": 5333
        },
        {
            "type": "mixed",
            "tag": "mixed-in",
            "listen": "::",
            "listen_port": 5330,
            "udp_timeout": "300s",
            "sniff": true,
            "sniff_override_destination": false,
            "set_system_proxy": false
        },
        {
            "type": "tun",
            "tag": "tun-in",
            "interface_name": "singtun0",
            "inet4_address": "172.19.0.1/30",
            "mtu": 9000,
            "gso": false,
            "auto_route": true,
            "udp_timeout": "300s",
            "stack": "system",
            "sniff": true,
            "sniff_override_destination": false
        }
    ],
    "outbounds": [
        {
            "type": "direct",
            "tag": "direct-out",
            "routing_mark": 100
        },
        {
            "type": "block",
            "tag": "block-out"
        },
        {
            "type": "dns",
            "tag": "dns-out"
        },
        {
            "type": "vless",
            "tag": "cfg-vless-out",
            "routing_mark": 100,
            "server": "你的域名",
            "server_port": 11111, // 端口
            "uuid": "服务端对应的uuid",
            "flow": "xtls-rprx-vision",
            "packet_encoding": "xudp",
            "tls": {
                "enabled": true,
                "insecure": false
            },
            "domain_strategy": "ipv4_only"
        },
        {
            "type": "trojan",
            "tag": "cfg-trojan-out",
            "routing_mark": 100,
            "server": "你的域名",
            "server_port": 22222, // 端口
            "password": "服务端对应的密码",
            "tls": {
                "enabled": true,
                "insecure": false
            },
            "domain_strategy": "ipv4_only"
        },
        {
            "type": "hysteria2",
            "tag": "cfg-hysteria2-out",
            "routing_mark": 100,
            "server": "服务端 ip",
            "server_port": 33333, //端口
            "password": "服务端对应的密码",
            "tls": {
                "enabled": true,
                "server_name": "你的域名",
                "insecure": false
            },
            "domain_strategy": "prefer_ipv4"
        },
        {
            "type": "tuic",
            "tag": "cfg-tuic-out",
            "routing_mark": 100,
            "server": "服务端 ip",
            "server_port": 44444, //端口
            "password": "服务端对应密码",
            "uuid": "服务端对应uuid",
            "congestion_control": "cubic",
            "heartbeat": "10s",
            "tls": {
                "enabled": true,
                "server_name": "你的域名",
                "insecure": false,
                "alpn": [
                    "h3"
                ]
            },
            "domain_strategy": "ipv4_only"
        },
        {
            "type": "selector",
            "tag": "select-out",
            "outbounds": [
                "cfg-vless-out",
                "cfg-trojan-out",
                "cfg-hysteria2-out",
                "cfg-tuic-out"
            ],
            "default": "cfg-hysteria2-out",
            "interrupt_exist_connections": true
        }
    ],
    "route": {
        "rules": [
            {
                "inbound": "dns-in",
                "outbound": "dns-out"
            },
            {
                "type": "logical",
                "mode": "or",
                "rules": [
                    {
                        "protocol": "dns"
                    },
                    {
                        "port": 53
                    }
                ],
                "outbound": "dns-out"
            },
            {
                "ip_is_private": true,
                "outbound": "direct-out"
            },
            {
                "rule_set": [
                    "cfg-ad-rule"
                ],
                "outbound": "block-out"
            },
            {
                "clash_mode": "Direct",
                "outbound": "direct-out"
            },
            {
                "clash_mode": "Global",
                "outbound": "select-out"
            },
            {
                "type": "logical",
                "mode": "or",
                "rules": [
                    {
                        "port": 853
                    },
                    {
                        "network": "udp",
                        "port": 443
                    },
                    {
                        "protocol": "stun"
                    }
                ],
                "outbound": "block-out"
            },
            {
                "rule_set": [
                    "geoip-cn",
                    "geosite-geolocation-cn"
                ],
                "outbound": "direct-out"
            },
            {
                "inbound": [
                    "mixed-in"
                ],
                "outbound": "cfg-hysteria2-out"
            },
            {
                "protocol": [
                    "quic"
                ],
                "outbound": "block-out"
            }
        ],
        "rule_set": [
            {
                "type": "remote",
                "tag": "geosite-geolocation-cn",
                "format": "binary",
                "url": "https://raw.githubusercontent.com/SagerNet/sing-geosite/rule-set/geosite-geolocation-cn.srs",
                "download_detour": "select-out"
            },
            {
                "type": "remote",
                "tag": "geosite-geolocation-!cn",
                "format": "binary",
                "url": "https://raw.githubusercontent.com/SagerNet/sing-geosite/rule-set/geosite-geolocation-!cn.srs",
                "download_detour": "select-out"
            },
            {
                "type": "remote",
                "tag": "geoip-cn",
                "format": "binary",
                "url": "https://raw.githubusercontent.com/SagerNet/sing-geoip/rule-set/geoip-cn.srs",
                "download_detour": "select-out"
            },
            {
                "type": "remote",
                "tag": "cfg-ad-rule",
                "format": "binary",
                "url": "https://github.com/TG-Twilight/AWAvenue-Ads-Rule/raw/main/Filters/AWAvenue-Ads-Rule-Singbox.srs",
                "download_detour": "select-out"
            }
        ],
        "auto_detect_interface": true,
        "final": "select-out"
    },
    "experimental": {
        "cache_file": {
            "enabled": true
        },
        "clash_api": {
            "external_controller": "192.168.31.4:9090", //你的机器  IP
            "external_ui": "ui",
            "external_ui_download_url": "https://github.com/MetaCubeX/Yacd-meta/archive/gh-pages.zip",
            "external_ui_download_detour": "select-out",
            "default_mode": "Rule"
        }
    }
}
```


